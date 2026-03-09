# Why Batch Flows Need Params

## The Problem

Batch flows need two kinds of data:

1. **Shared store**:mutable state that persists across all iterations. Every node in every iteration reads and writes to the same dict. Example: accumulating `results`.

2. **Per-iteration context**:different data for each iteration. Example: which `key` to process this time.

These have contradictory requirements. Shared must be **one mutable object**. Per-iteration context must be **different per iteration**. One dict cannot be both simultaneously.

## Running Example

A `BatchFlow` that doubles each value in a dict:

```python
# BatchFlow.prep returns per iteration params
class MyBatchFlow(BatchFlow):
    def prep(self, shared):
        return [{"key": k} for k in shared["data"]]  # [{"key":"a"}, {"key":"b"}]

# Node uses params to know which key to process, writes result to shared
class DoubleNode(Node):
    def prep(self, shared):
        return shared["data"][self.params["key"]]         # per iteration context
    def post(self, shared, prep_res, exec_res):
        shared["results"][self.params["key"]] = prep_res * 2  # mutate shared

shared = {"data": {"a": 1, "b": 2}, "results": {}}
MyBatchFlow(start=DoubleNode()).run(shared)
# Expected: shared["results"] == {"a": 2, "b": 4}
```

## Alternatives (and Why They Fail)

### Alternative 1: `shared.update(bp)`:mutate shared directly

```python
# Framework merges batch params into shared before each iteration
for bp in pr:
    shared.update(bp)     # shared["key"] = "a"
    self._orch(shared)    # node reads shared["key"]
```

**Works for sequential:**

```
Iteration 1: shared.update({"key": "a"}) → shared = {"data": ..., "results": {}, "key": "a"}
             node reads shared["key"] → "a", writes shared["results"]["a"] = 2  ✓
Iteration 2: shared.update({"key": "b"}) → shared["key"] = "b"
             node reads shared["key"] → "b", writes shared["results"]["b"] = 4  ✓
```

**Fails for parallel**:concurrent tasks overwrite each other:

```
Task 1: shared.update({"key": "a"})  → shared["key"] = "a"
Task 2: shared.update({"key": "b"})  → shared["key"] = "b"  (overwrites!)
Task 1: node reads shared["key"]     → "b"  ✗ (wanted "a")
Task 2: node reads shared["key"]     → "b"  ✓
```

Both tasks see `"b"`. Task 1's context is lost.

### Alternative 2: `{**shared, **bp}`:copy shared, merge bp in

```python
# Framework creates a new dict for each iteration
for bp in pr:
    self._orch({**shared, **bp})   # pass a copy with bp merged in
```

**Fails:mutations don't persist** across iterations:

```
Iteration 1: copy1 = {"data": {"a":1,"b":2}, "results": {}, "key": "a"}
             node reads copy1["key"] → "a"  ✓
             node writes copy1["results"]["a"] = 2
             But copy1 is a DIFFERENT dict from shared!
             shared["results"] is still {}  ✗

Iteration 2: copy2 = {"data": {"a":1,"b":2}, "results": {}, "key": "b"}
             copy2["results"] is a fresh {} (from shared, which was never modified)
             Previous result {"a": 2} is lost  ✗
```

After loop: `shared["results"]` is still `{}`. All writes went into throwaway copies.

### Alternative 3: `shared["_batch_queue"]`:queue in shared

```python
# Framework stores params as a queue; nodes pop from it
shared["_batch_queue"] = [{"key": "a"}, {"key": "b"}]
for _ in pr:
    self._orch(shared)
# Node does: bp = shared["_batch_queue"].pop(0)
```

**Works for flat batch flows.** But **fails when batch flows are nested:**

```
Outer BatchFlow prep → shared["_batch_queue"] = [{"group": "X"}, {"group": "Y"}]

Outer iteration 1:
  Inner node pops {"group": "X"} from shared["_batch_queue"]
  Inner BatchFlow prep → shared["_batch_queue"] = [{"item": 1}, {"item": 2}]  ← CLOBBERS outer queue!
  Inner processes items 1, 2... done.

Outer iteration 2:
  shared["_batch_queue"] is now [] (inner consumed it)
  {"group": "Y"} is gone forever  ✗
```

The inner BatchFlow overwrites `_batch_queue`, destroying the outer's remaining items. Group `"Y"` never gets processed.

## Why Params Work

Params provide a **second channel** scoped to one iteration:

```python
def _orch(self, shared, params=None):
    curr = copy.copy(self.start_node)
    while curr:
        curr.set_params(params)
        curr._run(shared)  # shared is the SAME dict, always
        ...
```

- **Shared stays shared**: one dict, passed by reference, mutations always visible everywhere.
- **Params are isolated**: each iteration sets its own params on node copies. No collision with shared keys, no collision between iterations, no collision between nesting levels.
- **Parallel safe**: each concurrent task passes different params to its own node copies. Shared is still the same dict:concurrent writes to different keys work fine.
- **Nest-safe**: inner batch flow sets params on its own nodes. Outer batch flow's params are on different node copies. No interference.

## The Fundamental Reason

The problem has **two dimensions**:

- **Horizontal (shared)**: data that flows across all nodes and all iterations. One mutable instance.
- **Vertical (params)**: data scoped to one iteration. Different per iteration, read-only from the framework's perspective.

Collapsing two dimensions into one dict always loses information. Params exist to address this:they keep batch-specific context separate from shared state.

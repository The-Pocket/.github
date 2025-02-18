<h1 align="center">Pocket Flow - Minimalist LLM Framework in 100 Lines</h1>

Table 1 compares PocketFlow to existing frameworks. The primary design choices are:

1. **Graph-based computation model:** PocketFlow does not adopt an *agent* oriented architecture.
2. **Shared store communication model:** Rather than message-passing, all nodes read and write from a *shared store*.
3. **No application- or vendor-specific modules:** PocketFlow focuses *exclusively* on core logic.
4. **Minimal footprint:** The entire codebase is *100 lines*, yielding a *56KB* distribution with *no external dependencies*.

|                | **Comput. Models**          | **Comm. Models**  | **App-Spec Models**                                      | **Vendor-Spec Models**                                    | **LOC**       | **Package + Dep. Size**    |
|----------------|-----------------------------|-------------------|-----------------------------------------------------------|------------------------------------------------------------|---------------|----------------------------|
| LangChain  | Agent, Chain               | Messages          | Many (e.g., QA, Summarization, etc.)                 | Many (e.g., OpenAI, Pinecone, etc.)                   | 405K          | +166MB                     |
| LlamaIndex | Agent, Graph               | Messages, Shared  | Native for RAG (Summarization, KG Indexing, etc.)        | Many [Optional] (e.g., OpenAI, Pinecone, etc.)        | 77K (core-only)   | +189MB (core-only)         |
| CrewAI     | Agent, Chain               | Messages, Shared  | Many (e.g., FileReadTool, SerperDevTool, etc.)       | Many (e.g., OpenAI, Anthropic, Pinecone, etc.)        | 18K           | +173MB                     |
| Haystack   | Agent, Graph               | Messages, Shared  | Many (e.g., QA, Summarization, etc.)                 | Many (e.g., OpenAI, Anthropic, Pinecone, etc.)        | 31K           | +195MB                     |
| SmolAgent   | Agent                      | Messages          | Some (e.g., CodeAgent, VisitWebpageTool, etc.)       | Some (e.g., DuckDuckGo, Hugging Face, etc.)           | 8K            | +198MB                     |
| LangGraph   | Agent, Graph               | Messages, Shared  | Some (e.g., Semantic Search, etc.)                   | Some (e.g., PostgresStore, SqliteSaver, etc.)         | 37K           | +51MB                      |
| AutoGen    | Agent                      | Messages          | Some (e.g., Tool Agent, Chat Agent, etc.)            | Many [Optional] (e.g., OpenAI, Pinecone, etc.)        | 7K (core-only)    | +26MB (core-only)          |
| **PocketFlow** | **Graph**                  | **Shared**        | **None**                                                 | **None**                                                  | **100**       | **+56KB**                  |

<p align="center"><small><em>Table 1: Comparison of AI system frameworks for computation models, communication models, application-specific models, vendor-specific models, lines of code, and package + dependency size. PocketFlow uses a nested directed graph (instead of agent or chain) as its computation model and shared storage (instead of message passing) as its communication model, without any application or vendor-specific models, requiring only 100 lines of code and a 56KB package + dependency size.</em></small></p>

## Computation Models

We use *graph* as the *computation model*. Each node represents a logical unit of work, while flows (graphs) orchestrate the interaction among nodes through directed edges. The direction indicates how data or control moves between nodes.

- **Node:** A minimal unit of computation (for example, an LLM call, tool use, or human feedback).
- **Flow:** A subgraph that coordinates nodes to accomplish a larger task.
- **Direction:** Edges are directed, indicating the flow of data or decisions (e.g., for agentic decisions).

Nodes in PocketFlow can also be: (1) **Nested:** One flow can be embedded in another, supporting hierarchical composition of logic. (2) **Batch:** Nodes can process multiple items in bulk. (3) **Async:** Tasks may run in parallel, pausing or waiting as needed.

### Why not use agent as the computation model?

1. **Expressibility:** Agents can be expressed as **graphs**, since typical agent behaviors (looping or branching) map directly to cyclical or conditional edges.
2. **Practicality:** Even for agents, the core design boils down to (1) context management and (2) action space, both of which are better handled via multi-step nodes rather than a standalone agent abstraction.

### Graph Representation:  Global or Local?

<table>
<tr>
<td><b>Global Graph Representation</b></td>
<td><b>Local Graph Representation</b></td>
</tr>
<tr>
<td>

```python
class Graph:
    def __init__(self):
        self.adj = {}  # adjacency map

# usage
g = Graph()
n3 = Node()
n4 = Node()
g.adj[n3] = [n4]
```

</td>
<td>

```python
class Node:
    def __init__(self):
        self.succ = []  # successors

# usage
n1 = Node()
n2 = Node()
n1.succ.append(n2)
```

</td>
<tr>
<td>
A separate <i>Graph</i> maintains a global adjacency map.
</td>
<td>
Each individual <i>Node</i> tracks its successors.
</td>
</tr>
</table>
<p align="center"><small><em>Table 2: Comparison of Global and Local Graph Representation.</em></small></p>


Another key design choice is how to represent the graph (nodes + edges). *LangGraph* adopts a *global* graph representation: once a graph is compiled, it becomes a finalized structure that cannot be easily modified. By contrast, **PocketFlow chooses a local graph representation**, a design shared by popular workflow systems like [Airflow](https://airflow.apache.org/), [Luigi](https://luigi.readthedocs.io/en/stable/), and [Prefect](https://www.prefect.io/). This approach ensures *modularity*: because each subgraph is self-contained, making it easier to understand and maintain in isolation. Moreover, *incremental development* is simpler, since new pipelines can be introduced or updated without affecting other flows.

## Communication Models

Existing frameworks let compute units communicate majorly via **message passing**, some also maintain a global context as a back door. In contrast, **PocketFlow uses solely a shared store as the communication model**, where each node reads and writes to the shared store. This design choice of shared store is for **separation of concerns**:

- **Data Structure** is designed and maintained in a central place.
- **Compute logic** is then operated on the defined data structure.

While *message passing* can be simple for smaller systems, it becomes cumbersome to maintain with many interacting components. A *shared store* allows developers to modify data structures freely, without refactoring communication patterns among nodes.

## Application or Vendor-Specific Models

PocketFlow does not include modules that target particular vendors or use cases. While this may seem to forgo certain benefits of *information hiding*, it keeps the framework *minimal* and avoids shifting dependencies. Modern LLM systems often require integrating various APIs (for LLM calls or tool usage), which are challenging to maintain in a single framework.

From our perspective, the job of handling vendor-specific details is best left to AI assistants that can retrieve up-to-date documentation and assemble relevant code on the fly. **PocketFlow remains a lightweight core, while an AI assistant—like Pocket AI—provides tailored integrations for specific business or application needs.**

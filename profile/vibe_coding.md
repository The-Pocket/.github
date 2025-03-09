<h1 align="center">Vibe Coding with Pocket Flow</h1>

<!-- I Tried to "Vibe Code" AI Agents with My 100-line Framework—and It Actually Worked! (Video Walkthrough) -->

I created a video tutorial demonstrating how I used Cursor AI to "vibe code" an AI Paul Graham: https://youtu.be/Cf38Bi8U0Js. The secret weapons is a [100-line LLM framework](https://github.com/The-Pocket/PocketFlow) \+ [`cursorrules`](https://github.com/The-Pocket/PocketFlow/blob/main/.cursorrules) to enforce Cursor AI to write modular and maintainable codes.

## What's Vibe Coding?

It's a term coined by [Andrej Karpathy](https://x.com/karpathy/status/1886192184808149383). It's a programming practice where humans specify what to build, and an AI (like Cursor) handles the low-level implementation. This approach >10x development productivity.

Since Cursor's release, I've become a big fan of code automation, especially for building AI agents. However, previously it didn't work well. The main issue wasn't Cursor but rather the LLM framework.

## Issues with the Current Framework

1. **Vendor Wrapper Overload**
    - *❤️ Ideally*: Provide an all-in-one toolbox so developers just state what they need, and everything is provided.
    - *💔 In Reality*: Large package sizes (100–200 MB), massive codebases (10K–400K lines), installation dependency issues with unnecessary packages, and vendor lock-in.

2. **Application Wrappers Confusion**
    - *❤️ Ideally*: Provide application wrappers for common use cases, making it easy for developers to get started quickly.
    - *💔 In Reality*: Overlapping wrappers, outdated or incomplete documentation, and frequent deprecations, leading to confusion.

These issues confuse not only humans but also Cursor. As a result, it attempts to call non-existing functions and eventually gives up on the framework.

## What About No Framework?

Ironically, using no framework often produces more functional code initially but quickly becomes a **maintenance nightmare**:

- When you ask Cursor AI to implement something, it attempts to finish the task in the most convenient way possible, without fully understanding existing functions and modules. This results in lots of **ad hoc** and **one-shot** code.
- As projects grow, the codebase becomes increasingly confusing. You end up spending about 10% of your time vibe coding and 90% vibe debugging.

## How Does My 100-Line Framework Help?

Consider my [100-line LLM framework](https://github.com/The-Pocket/PocketFlow) as **LangGraph** but with only the core graph abstraction—no vendor-specific or application-level logic.

- **Why a Graph?** I've found that the graph abstraction (1) is extremely expressive for various designs (e.g., ([multi-](https://the-pocket.github.io/PocketFlow/design_pattern/multi_agent.html))[agents](https://the-pocket.github.io/PocketFlow/design_pattern/agent.html), [Retrieval-Augmented Generation (RAG)](https://the-pocket.github.io/PocketFlow/design_pattern/rag.html), [workflow](https://the-pocket.github.io/PocketFlow/design_pattern/workflow.html), etc.) and (2) enforces excellent [modularity](https://the-pocket.github.io/PocketFlow/core_abstraction/node.html). Consequently, Cursor's "vibed code" is easy to maintain.

- **What about vendor-specific or application-level logic?**
  I've moved that logic into [documentation](https://the-pocket.github.io/PocketFlow/) provided to Cursor as [`cursorrules`](https://github.com/The-Pocket/PocketFlow/blob/main/.cursorrules). I've also included the system design [best practice](https://the-pocket.github.io/PocketFlow/guide.html). Cursor uses these rules during in-context learning to adapt to new applications, keeping the core framework minimal.

- **The Result? Great Vibes!** Cursor AI (or any LLM tool) quickly grasps the 100 lines of framework code and adapts it to various application requirements. The resulting code is modular and maintainable. It's hard to capture the vibe through words alone, so check out my [Video Walkthrough](https://youtu.be/Cf38Bi8U0Js).

## Future of Vibe Coding

- **Design Your Project Structure for Modularity and Testability**: One reason I love the graph approach is its built-in modularity. Cursor AI can build individual nodes, allowing easy isolation and troubleshooting of problems. You can let Cursor AI implement a node, test it independently, and proceed in manageable steps.

- **System Design Outranks Coding**: This aligns with standard software engineering career progression, where junior engineers handle coding, and senior engineers focus on architectural design. Cursor can automate most low-level coding, but humans should focus on high-level designs to make sure are easy to implement and maintain.


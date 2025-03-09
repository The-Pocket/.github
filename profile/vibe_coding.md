<h1 align="center">Vibe Coding with Pocket Flow</h1>

<!-- I Tried to "Vibe Code" AI Agents with My 100-line Framework—and It Actually Worked! (Video Walkthrough) -->

I created a video tutorial demonstrating how I used Cursor AI to "vibe code" an AI Paul Graham: https://youtu.be/wc9O-9mcObc

## What's Vibe Code?

Vibe coding is a term coined by [Andrej Karpathy](https://x.com/karpathy/status/1886192184808149383). It's a programming practice in which humans specify what to build, and an AI (e.g., Cursor) handles low-level implementation. This approach can make development more than ten times as productive.

Since the release of Cursor, I’ve become a big fan of code automation, especially for building AI agents. However, it didn't work well previously. The main issue wasn't Cursor but rather the LLM framework.

## Issues with the Current Framework

- 1. **Vendor Wrapper Overload**
    - *❤️ Ideally*: Provide an all-in-one toolbox so developers just states what they need, and everything is provided.
    - *💔 In Reality*: Large package sizes (100–200 MB), massive codebases (10K–400K lines), installation dependency issues for packages aren’t even needed, and vendor lock-in.

2. **Application Wrappers Confusion**
    - *❤️ Ideally*: Provide application wrappers for common use cases, so it’s easy for developers to get started quickly.
    - *💔 In Reality*: Overlapping wrappers, outdated or incomplete documentation, and deprecation all lead to confusion.

These not only confuse humans but aslo Cursor. As a result, it attempts to call nonexisting functions, and eventually gives up on the framework.

## What about no framework?

Ironically, using no framework often works better in terms of producing functional code. However, it becomes a **maintenance nightmare**:

- When you ask Cursor AI to implement something, it tries to finish using the most convenient way as possible, without understanding existing functions and modules. As a result, they create a lot of **ad hoc** and **one-shot** codes.
- As projects grow, the codebase becomes highly confusing. You end up spending about 10% of your time vibe coding and 90% vibe debugging.


## How Does My 100-Line Framework Help?

Consider my framework as **LangGraph** but with only the core graph abstraction—no vendor-specific or application-level logic.

- **Why a Graph?** I've found that graph abstraction (1) is extremely expressive for all kinds of designs (e.g., agents, retrieval-augmented generation, workflows, etc.) and (2) enforces excellent modularity. The result is that the "vibed codes" by Cursor is easy to maintain.

- **What about vendor-specific or application-level logic?**
  I've moved that logic into documentation that I provide to Cursor as `cursorrule`. Cursor can use these rules during in-context learning to adapt to new applications, while the core framework remains minimal.

- **The Result? Great Vibes!** Cursor AI (or any LLM tool) can quickly grasp the 100 lines of framework code and adapt them to meet various application requirements. The code is also modular and maintainable. It's hard to pass the vibe through words but check out my Video Walkthrough.

## Future of Vibe Coding
 
 - **Design Your Project Structure for Modularity and Testability**: One reason I love the graph approach is that it provides modularity by default. Cursor AI can build individual nodes, and if something goes wrong, you can isolate the problem to the right node without affecting everything else. You can let Cursor AI implement a node, test it, and proceed in small steps.

- **System Design Outranks Coding**: This aligns with standard software engineering career ladders, where junior engineers handle coding and senior engineers focus on design architecture. Cursor can automates most loew-level coding, and human's focus should be on the more high level designs in a way that are easy to implement.


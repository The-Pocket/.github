<h1 align="center">Pocket AI - Designing Systems, Not Just Writing Code</h1>

*(This essay is in progress)*

## System Design, Not Just Coding

Although AI software engineers are in high demand, the core ability lies in designing systems, not just writing code. This includes deciding how to structure the codebase, identifying the core components, keeping the system maintainable, creating testable code, ensuring scalability, and establishing a solid CI/CD process.

## AI Interfaces: Chat-based

There are multiple ways to interact with AI:

- **Chat for single-turn interactions**: When ChatGPT first launched, it provided a simple chat interface. This ease of use led to rapid growth, with 100 million monthly users in just two months. Although chat works well for straightforward, single-turn interactions, it isn't ideal for more complex enterprise use cases.

- **Drag-and-drop for complex systems**: More advanced AI solutions often require more than a single LLM call. Enterprise use cases typically involve multiple components, error handling, testing, and retries. To manage these complexities, low-code/no-code tools like [Flowise](https://flowise.ai/), [LangFlow](https://langflow.com/), [Gumloop](https://gumloop.com/), and [n8n](https://n8n.io/) allow users to build workflows through drag-and-drop interfaces.

- **Chat, but for complex systems**: Despite the rise of specialized interfaces, we predict a renewed focus on chat-based tools that also allow code editing. Chat remains highly user-friendly, and incorporating code helps maintain flexibility and reliability. Platforms like [cursor.ai](https://www.cursor.com/) and [bolt.new](https://bolt.new/) show this in action. We expect more solutions that combine the simplicity of chat with the robustness of coding support.

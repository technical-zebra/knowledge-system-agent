# Personal AI System Architecture

This document describes the architecture of the **Personal AI System (Second Brain)**, also known as agent "DANNER", which automatically manages a knowledge base and interacts with external tools.

## Architecture Layers

The architecture is divided into the following key layers, from the foundation up to the remote access layer:

### 1. Knowledge Base - Foundation
The core layer that stores and organizes all information.
* **Articles:** The base knowledge consisting of cleaned data from over 4000+ ChatGPT conversations.
* **Knowledge Graph (Concept Relations):** Organizes the articles by clarifying the relationships and structures between different concepts.
* **AI Dictionary (Entity Terms):** A byproduct created while building the Knowledge Graph. It serves as an extracted list of key AI concepts and technical terms with their contexts and explanations. It is compatible with the Jieba tokenization format, making it easy to plug into existing pipelines.

### 2. Update Pipeline
An automated, continuous ingestion pipeline that feeds new knowledge into the foundation.
* **Ingestion Sources:**
  * Reddit
  * X (formerly Twitter)
  * YouTube
  * New AI Conversations
These sources constantly update the AI Dictionary and the Knowledge Graph, ensuring the knowledge base remains current.

### 3. Skills - Tools Layer
The active layer where the AI connects to various tools to perform tasks based on the knowledge base.
* **Content Gen:** General content generation.
* **IndexTTS2:** Text-to-speech generation.
* **Remotion:** Video generation.
* **LinkedIn:** Automated LinkedIn post creation.
* **Blog/SEO:** SEO-friendly blog post generation.
* **Claude Code SDK:** Integrated via OpenClaw as a specialized AI coding skill. This delegates complex coding, refactoring, and tool-calling workflows directly to the Claude Code CLI, significantly saving token costs and leveraging Claude's native plugins (workflows) and terminal capabilities.
* **oh-my-opencode:** A productivity framework and plugin ecosystem for managing AI coding environments and workflows.
* *Extensible:* Capable of connecting with any custom skills to boost productivity.

### 4. Remote Access Layer
The outermost layer that provides the user interface for interacting with the AI system.
* **OpenClaw (Gateway):** Acts as the runtime/interaction gateway.
* **Discord:** Connected via OpenClaw, allowing you to access and call the entire AI system remotely from anywhere using Discord.
* **Feishu (Lark):** Connected via OpenClaw, enabling enterprise-level remote communication and task delegation within your workspace.

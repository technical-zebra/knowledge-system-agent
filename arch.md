# Personal AI System Architecture

This document describes the architecture of the **Personal AI System (Second Brain)**, also known as agent "DANNER", which automatically manages a knowledge base and interacts with external tools.

## Architecture Layers

The architecture is divided into the following key layers, from the foundation up to the remote access layer:

### 1. Knowledge Base - Foundation
The core layer that stores and organizes all information.
* **Articles:** The base knowledge consisting of cleaned data from over 4000+ ChatGPT conversations.
* **Knowledge Graph (Concept Relations):** Organizes the articles by clarifying the relationships and structures between different concepts. Powered by **LlamaIndex** as the primary orchestration engine for GraphRAG and indexing.
* **AI Dictionary (Entity Terms / ai-glossary-zh):** A byproduct created while building the Knowledge Graph. It serves as an extracted bilingual (Chinese-English) list of key AI concepts and technical terms. Initially extracted from 924 AI-related articles using LLM (GPT-4o-mini) filtering, it holds over 265 structured entries categorized into types (concept, company, product, framework, platform). Integrated via LlamaIndex for:
  * **Chinese Word Segmentation:** Fully compatible with the `jieba` tokenization format via `dict.txt`.
  * **Term Normalization:** Merges multiple aliases and Chinese translations (e.g., "向量嵌入", "嵌入") into canonical English terms (e.g., "embedding") via `glossary.json`.
  * **Translation Reference:** Provides standardized terminology for text parsing and downstream RAG indexing.
  * **Project Link:** https://github.com/SabrinaCatpenter/ai-glossary-zh

### 2. Update Pipeline
An automated, continuous ingestion pipeline that feeds new knowledge into the foundation. Highly optimized for cost-effectiveness.
* **Ingestion Sources:**
  * **Reddit:** Utilizing `PRAW` (Python Reddit API Wrapper) for inexpensive/free metadata and comment extraction.
  * **X (formerly Twitter):** Utilizing third-party scrapers (e.g., Apify Tweets-X-Scraper or open-source libraries) to bypass prohibitive official API costs via scheduled batch runs.
  * **YouTube:** Utilizing `youtube-transcript-api` and LangChain's `YoutubeLoader` to extract chunked textual transcripts directly, negating the need for expensive video processing.
  * **New AI Conversations:** Utilizing community parsing scripts (e.g., `slyubarskiy/chatgpt-conversation-extractor` and `claude-conversation-extractor`) to parse heavy, complex JSON exports into clean Markdown for indexing.
These sources utilize a lightweight Python ETL workflow (e.g., `dlt` or serverless cron jobs) to constantly update the AI Dictionary and the Knowledge Graph, ensuring the knowledge base remains current.

### 3. Skills - Tools Layer
The active layer where the AI connects to various tools to perform tasks based on the knowledge base.
* **Content Gen:** General content generation.
* **IndexTTS2:** Text-to-speech generation.
* **Remotion:** Video generation.
* **LinkedIn:** Automated LinkedIn post creation.
* **Blog/SEO:** SEO-friendly blog post generation.
* **Claude Code SDK (via Agent SDK):** The full Claude Code capability packaged as a programmable library by Anthropic. Core architecture:
  * **Agent Loop:** Claude auto-reasons → calls tools → observes results → continues reasoning, all within a single local process.
  * **Built-in Tools:** File read/write (`Read`, `Edit`, `Write`), Bash command execution (`Bash`), code search (`Glob`, `Grep`) — all available out-of-the-box.
  * **Session Management:** Each chat maps to an independent Agent Session via `chat_id`. Sessions support `resume: sessionId` for multi-turn context continuity.
  * **Non-Isolation Mode:** By configuring `settingSources: ["user", "project", "local"]`, the SDK loads `.CLAUDE.md` project instructions, `.claude/settings.json` MCP Server configs, and `.claude/skills/` custom skills — inheriting the full Claude Code ecosystem.
  * **Streaming Output:** Real-time `onUpdate` callbacks push Agent progress incrementally (file reads, command executions, reasoning steps) instead of waiting for full completion.
  * **Session Persistence:** All sessions serialized to a local JSON file (`.bot-sessions.json`), surviving bot restarts. Sessions include `sessionId`, `label` (first 30 chars of prompt), and `lastUsed` timestamp.
  * **Environment Config:** Requires explicit model variables (`ANTHROPIC_MODEL`, `ANTHROPIC_SMALL_FAST_MODEL`, etc.) when using third-party API endpoints. `env` must spread `process.env` to avoid losing system PATH.
* **oh-my-opencode:** A productivity framework and plugin ecosystem for managing AI coding environments and workflows.
* *Extensible:* Capable of connecting with any custom skills to boost productivity.

### 4. Remote Access Layer & Command Interface
The outermost layer that provides the user interface for interacting with the AI system. Commands can trigger specific skill modalities dynamically.
* **OpenClaw (Gateway):** Acts as the runtime/interaction gateway. It supports distinct operational modes triggered by simple chat commands:
  * `/omoc` or `/code` mode: Activates the **oh-my-opencode** framework, dropping you into a multi-agent multi-turn coding mode seamlessly without switching to a PC.
  * `/claude` mode: Directly opens the **Claude Code SDK** session within the chat interface for precise terminal-level source code modifications and workflows.
  * `/sessions` : Lists all persisted Agent Sessions, allowing quick context switching.
  * `/new` : Creates a fresh Agent Session.
  * `/reset` : Clears the current session.
  * `[Natural Language]`: Standard conversational mode for querying the local memory and LlamaIndex knowledge base.
* **Discord:** Connected via OpenClaw, allowing you to access and call the entire AI system remotely from anywhere using Discord.
* **Feishu (Lark):** Primary mobile remote access platform, connected via OpenClaw with adapter pattern. Key capabilities:
  * **Streaming Output:** Agent progress is pushed in real-time via Feishu interactive card `message.patch` — each step (file reads, command runs, reasoning) updates the card in-place.
  * **Markdown Rendering:** All Agent output rendered via Feishu card `tag: "markdown"` elements, providing proper code blocks, bold/italic formatting, and clickable links.
  * **Session Switching:** `/sessions` displays a card list of all persisted sessions; user replies with a number to switch (using `awaitingSwitch` state pattern, no HTTP callback server needed).
  * **Menu Support:** Feishu bot menu configured for quick command access (`/new`, `/sessions`, `/reset`, `/mode`).
  * **Architecture:** WebSocket long-poll mode (no server required), adapter pattern enables shared Agent core with minimal platform-specific code (~250 lines).

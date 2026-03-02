# OpenClaw Deploy Ninja 🥷🦞

**The ultimate zero-dependency, bulletproof AI Agent setup skill for OpenClaw.**

This repository contains a specialized AI Agent Skill designed to flawlessly deploy, isolate, and configure [OpenClaw](https://openclaw.ai) across any macOS or Linux environment.

## 🌟 Why use this Skill?
OpenClaw is a highly capable AI agent framework, but configuring it robustly requires system-level knowledge. This skill acts as your personal DevOps engineer, executing a "clean room" installation:

- **Zero Dependencies**: You don't need Node.js installed. It dynamically fetches architecture-specific runtimes (Intel/ARM64) and bundles them inside the app directory.
- **Mac External Drive Immunity**: Natively bypasses macOS `launchd`'s notorious `Exit Code 78` security block when installing to an external USB/SSD drive (`/Volumes/...`).
- **Interactive Model Tiering**: Automatically interviews the user to handle 3 distinct scenarios:
  1. **OAuth Freeloaders**: Guides users through Google Gemini CLI / OpenAI Codex web-based authentication to use AI for free.
  2. **API Whales**: Native setup for official Anthropic/OpenAI keys.
  3. **Proxy Users (小白福音)**: Automatically generates complex JSON fallbacks to seamlessly utilize third-party aggregators (e.g., NewAPI, OneAPI).

## 🚀 How to Use

1. Drop the `openclaw-setup.md` file into your OpenClaw or OpenCode skills directory.
2. In your chat interface, simply type:
   > *"Use the openclaw-setup skill to install OpenClaw for me."*
3. The AI Agent will take over, interview you about your drive preferences and API keys, and hand you a fully configured Dashboard URL within 30 seconds.

## 📂 Project Structure
- `openclaw-setup.md`: The core AI Skill definition file written in markdown. Simply import this into your agent's context.

---
*Built from the trenches of debugging macOS daemons and SSRF network proxies. Stop fighting the terminal, let the Agent deploy the Agent.*

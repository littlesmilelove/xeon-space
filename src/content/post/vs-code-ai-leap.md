---
title: "VS Code's AI Leap: Agent Mode, MCP Integration & BYOK Freedom Arrive in v1.99"
description: "Explore the powerful new AI capabilities in VS Code v1.99, including the integrated Agent Mode, flexible MCP server support, and Bring Your Own Key (BYOK) for ultimate AI model choice."
publishDate: "2025-04-08"
tags: ["VS Code", "AI", "Agent Mode", "MCP", "MCP Server", "BYOK"]
---

# VS Code v1.99 Update: Agent Mode, MCP Server Support, and BYOK

VS Code has rolled out a significant update, version 1.99 ([https://code.visualstudio.com/updates/v1_99](https://code.visualstudio.com/updates/v1_99)). This release introduces major features like Agent mode, Model Context Protocol (MCP) server support, and Bring Your Own Key (BYOK) (Preview).

## Agent Mode

You can enable Agent mode via the `chat.agent.enabled` setting. The experience is similar to `Cline` or `Cursor`, but being built directly into VS Code, it offers a smoother and more tightly integrated workflow. Additionally, VS Code includes several built-in tools for Agent mode:

### Thinking Tool (Experimental)

*   Enable this via the `github.copilot.chat.agent.thinkingTool` setting.
*   This tool allows AI models to perform intermediate "thinking" steps during tool execution, potentially improving performance on complex tasks.

### Fetch Tool

*   Use the `#fetch` tool to retrieve content from public web pages directly within the chat.
*   This eliminates the need for separately configured Search-related MCP Servers.

### Usages Tool

*   The `#usages` tool combines the functionality of "Find All References", "Find Implementation", and "Go to Definition".
*   For example, the chat agent can use this tool to look for sample implementations of an interface or to find all locations that need modification during a refactoring.

## Model Context Protocol (MCP) Server Support

VS Code now allows adding various MCP Servers using the `MCP: Add Server` command. A key differentiator is VS Code's flexibility in configuring MCP server scope: globally, per Workspace, for remote connections, or within a `.code-workspace` file.

Furthermore, by enabling the `chat.mcp.discovery.enabled` setting, VS Code can automatically discover MCP servers configured in other tools, like Claude Desktop.

You can see the list of configured MCP servers and their current status using the `MCP: List Servers` command. In agent mode, you can pick the specific tools available for use in chat by using the "Select Tools" button.

## Bring Your Own Key (BYOK) (Preview)

Copilot Pro and Copilot Free users can now configure their own API keys to utilize corresponding AI models. Currently supported providers include:

*   Azure
*   Anthropic
*   Gemini
*   OpenAI
*   Ollama
*   Open Router

This provides users with significantly more flexibility. However, it is currently unclear whether using your own key consumes the request quota associated with your Copilot Pro or Copilot Free subscription. The official documentation does not yet clarify this point.
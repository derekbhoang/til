# Model Context Protocol - MCP 

## MCP Architecture Overview

The Model Context Protocol follows a client-host-server architecture: This separation of concerns allows for modular, composable systems where each server can focus on a specific domain (like file access, web search, or database operations).

- **MCP Host:** The AI application that coordinates and manages one or multiple MCP clients.
- **MCP Client:** A component that maintains a connection to an MCP server and obtains context from an MCP server for the MCP host to use.
- **MCP Server:** A program that provides context, such as tools, resources (data), or prompts to MCP clients.

```mermaid theme={null}
graph TB
    subgraph "MCP Host (AI Application)"
        Client1["MCP Client 1"]
        Client2["MCP Client 2"]
        Client3["MCP Client 3"]
        Client4["MCP Client 4"]
    end

    ServerA["MCP Server A - Local<br/>(e.g. Filesystem)"]
    ServerB["MCP Server B - Local<br/>(e.g. Database)"]
    ServerC["MCP Server C - Remote<br/>(e.g. Sentry)"]

    Client1 ---|"Dedicated<br/>connection"| ServerA
    Client2 ---|"Dedicated<br/>connection"| ServerB
    Client3 ---|"Dedicated<br/>connection"| ServerC
    Client4 ---|"Dedicated<br/>connection"| ServerC
```
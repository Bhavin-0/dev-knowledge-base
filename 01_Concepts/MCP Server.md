---
type: concept
created: 2026-03-30
updated: 2026-03-30
status: learning
tags: []
topic: system-design
reviewed:
difficulty:
source:
---

# MCP Server

## Definition
It gives access to set of tools. It exposes funcitonalities of some outside service.

MCP (Model Context Protocol) = Servers that give AI models contolled access to tools/data.

Safe Middleware systems between AI and real systems

---

## What it does
- AI asks MCP -> get data/ do action.
- MCP servers exectues 
- Returns results to AI

---

## How It Works
![[Pasted image 20260411003128.png]]

# Simple Visual Flow 
```
User
  ↓
Your Server (backend)
  ↓
MCP Client (SDK inside your server)
  ↓
MCP Server (tool provider)
  ↓
External Tool/API (GitHub, DB, etc)

⬆ response comes back same way ⬆
```
---
# What each part is ?
**Your Server : ** Controls everything
**MCP Client: ** Communicator (talks MCP protocols) -> what tools are required ?
**MCP Server: ** exposes tools  -> available tools
**Tools/API: ** does the real work


---

## Example
User:

> “Show my GitHub repos”

Flow:

1. Server gets query
2. MCP client asks: “what tools exist?”
3. MCP server: “getRepos tool available”
4. Server sends query + tools to AI
5. AI: “call getRepos”
6. MCP client → MCP server → GitHub API
7. Data comes back
8. AI formats answer
9. User gets response

---

## Advantages / Limitations

Advantages:
- 

Limitations:
- 

---

## Commands / Code

```bash

```
---
## Status
- [ ] Not understood
- [ ] Partially understood
- [ ] Clear

---
## MOC
```dataview
table link("00_Index/" + topic, topic) as "MOC"
where file.name = this.file.name
```

[[00_Index/system-design]]

---

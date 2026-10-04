# Discord-Quest-Completer
<div align="center">

# 💠 Lingy.dev · Discord Quest Script

**Game internals. Custom tooling. Clean code.**

A JavaScript utility for simulating activity and reporting progress across supported Discord Quests.

`JavaScript` · `Discord Client` · `Quest Automation`

</div>

---

## ⚡ Overview

The script detects enrolled, unfinished, unexpired quests and attempts to process supported tasks sequentially. Depending on the task, it sends progress requests or temporarily overrides client activity data.

## 🧩 Supported Task Types

| Task | Implementation |
|:--|:--|
| 🎬 **Watch Video** | Sends video progress updates |
| 📱 **Watch on Mobile** | Handles the mobile video task type |
| 🎮 **Play on Desktop** | Simulates a running game |
| 📡 **Stream on Desktop** | Overrides stream metadata |
| 🕹️ **Play Activity** | Sends activity heartbeat requests |

## 🔧 Under the Hood

- Finds internal client modules through Discord’s Webpack runtime.
- Reads quest requirements and existing progress.
- Logs progress updates to the console.
- Restores overridden game and stream methods after successful completion.

## 📌 Technical Notes

The script depends on undocumented Discord internals. Client updates may break module discovery or task handling.

Desktop play and streaming branches require the desktop client. Errors can interrupt the queue, and cleanup after a failed run is not guaranteed. Completing progress does not automatically claim a reward.

## 💻 Source Code

Place the JavaScript file alongside this README as **`script.js`**.

[**View script →**](./script.js)

---

<div align="center">

**Lingy.dev**  
Development · Reverse Engineering · Custom Tools

</div>

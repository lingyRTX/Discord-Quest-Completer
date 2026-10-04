<div align="center">

# 💠 Lingy.dev · Discord Quest Script
<img width="931" height="523" alt="image" src="https://github.com/user-attachments/assets/ec11c8a9-dc2e-457a-8183-b6f5a736636e" />
**Game internals. Custom tooling. Clean code.**

A JavaScript utility for simulating activity and reporting progress across supported Discord Quests.

`JavaScript` · `Discord Client` · `Quest Automation`

[Explore the script](./script.js) · [Supported tasks](#-supported-task-types) · [Technical notes](#-technical-notes)

</div>

---

## ⚡ Overview

The script detects enrolled, unfinished, unexpired quests and attempts to process supported tasks sequentially. Depending on the task, it sends progress requests or temporarily overrides client activity data.

This repository contains a standalone script, with no build step or package dependencies.

## 🧩 Supported Task Types

| Task | Identifier | Implementation |
|:--|:--|:--|
| 🎬 **Watch Video** | `WATCH_VIDEO` | Sends video progress updates |
| 📱 **Watch on Mobile** | `WATCH_VIDEO_ON_MOBILE` | Handles the mobile video task type |
| 🎮 **Play on Desktop** | `PLAY_ON_DESKTOP` | Simulates a running game |
| 📡 **Stream on Desktop** | `STREAM_ON_DESKTOP` | Overrides stream metadata |
| 🕹️ **Play Activity** | `PLAY_ACTIVITY` | Sends activity heartbeat requests |

These are branches implemented in the source, not a verified compatibility list.

## 🔧 Under the Hood

1. Obtains access to the client's Webpack module registry.
2. Locates internal quest, game, stream, channel, dispatcher, and API modules.
3. Filters quests by enrollment, completion status, expiration, and task type.
4. Reads the selected task's target and existing progress.
5. Runs the corresponding activity simulation or progress request loop.
6. Logs progress and moves to the next quest after the current branch completes.

For desktop game and stream tasks, overridden methods are restored after the expected completion event.

## 📂 Repository Contents

```text
.
├── README.md     Project overview and implementation notes
└── script.js     Original JavaScript source
```

## 💻 Source Code

[**Open script.js →**](./script.js)

The script is intended for Discord's client JavaScript environment. It is not a Node.js command-line program, a browser extension, or a Discord bot.

## 📋 Console Output

Examples of messages defined in the source:

```text
You don't have any uncompleted quests!
Quest progress: <current>/<target>
Quest completed!
```

Console messages describe the script's execution. They are not an independent verification that a reward was granted.

## 📌 Technical Notes

### Client compatibility

The script relies on undocumented Discord internals, including module exports, store methods, event names, and request formats. Client updates may break these assumptions.

The desktop play and streaming branches check for the desktop client. Handling a mobile video task identifier does not mean the script runs in the mobile app.

### Queue behavior

Quests are collected once at startup and processed using an array queue. Newly enrolled quests are not added during execution. If a quest contains multiple supported task types, the script selects the first matching type from its configured list.

Some unsupported-environment branches stop without advancing the queue. Unhandled request or lookup errors can also interrupt execution.

### Cleanup

Game and stream method overrides are restored on successful completion. The source does not provide comprehensive error handling or guaranteed cleanup after failure or interruption.

### Progress and rewards

The script attempts to report quest progress. It does not include a separate reward-claim operation, and successful completion or reward eligibility is not guaranteed.

## 💬 Contact

Find me on Discord: **`@lingy.`**

## 🧪 Verification Status

This documentation was prepared by reading the supplied source. The script was not executed or tested against a live Discord client, and compatibility with current client versions has not been verified.

---

<div align="center">

**Lingy.dev**

Development · Reverse Engineering · Custom Tools

</div>

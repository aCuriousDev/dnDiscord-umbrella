# Glossary - DnDiscord

DnDiscord is a Dungeons and Dragons campaign manager that runs as a Discord Activity - a full-screen iframe embedded inside a Discord voice channel. This glossary explains the domain and technical terms a new contributor or evaluator will encounter, with pointers to the code where each concept lives.

---

## Game / D&D

- **DM (Dungeon Master)** - The player who runs the game: controls enemies, narrates events, manages the map, and grants rewards. In the codebase the DM is tracked via `Campaign.DungeonMasterId` and is explicitly not a `CampaignMember`; hub role checks distinguish them from players. _Code:_ `back/Multiplayer/Define.cs:PlayerRole.DungeonMaster`

- **Player** - Any participant who is not the DM. Players control one character token on the board, take turns in combat, and can receive items and XP from the DM. _Code:_ `back/Multiplayer/Define.cs:PlayerRole.Player`

- **Character** - A persistent D&D character owned by a player, stored in the database. Holds stats, class, level, HP, inventory, and wallet. In a session the character is converted into a board unit via `CharacterToUnit`. _Code:_ `back/src/DnDiscordAPI/Games/` and `front/dndiscord-esp/src/game/__tests__/CharacterToUnit.test.ts`

- **Class** - The D&D archetype chosen at character creation (warrior, mage, archer in the quickstart presets). Determines abilities, hit dice, and base stats. Purely a data field on the character entity; no class-specific server logic in the POC.

- **Encounter** - A structured combat event started by the DM via `DmStartCombat`. The server rolls initiative, assigns a turn order, and transitions the session phase from `FreeRoam` to `Preparation` and then `PlayerTurn` / `EnemyTurn`. _Code:_ `back/Multiplayer/Services/CombatManager.cs`

- **Turn** - One unit's action window inside an encounter. The active unit may move and spend AP; calling `EndTurn` advances to the next unit in the initiative order. _Code:_ `back/Multiplayer/Services/TurnManager.cs`, `front/dndiscord-esp/src/services/signalr/turnEndedLogic.ts`

- **Round** - One full cycle through every unit in the `TurnOrder`. The server increments `CombatState.Round` each time the cursor wraps. _Code:_ `back/Multiplayer/Models/CombatState.cs`

- **Initiative** - The order in which units act during an encounter, determined by a D20 roll at the start of combat. The sorted list is stored in `CombatState.TurnOrder`. _Code:_ `back/Multiplayer/Models/CombatState.cs:TurnOrder`

- **ASI (Ability Score Improvement)** - A bonus granted at certain levels allowing a player to raise one or more ability scores. The server flags a pending ASI after a qualifying level-up; `DmForceLevelUp` can apply it explicitly. _Code:_ `back/src/DnDiscordAPI/Games/Character/Services/CharacterProgressionAdapter.cs`

- **Level-up** - The event triggered when a character's XP crosses a threshold. The server applies it automatically inside `CharacterProgressionAdapter` and broadcasts `CharacterProgressed` to the session. _Code:_ `back/src/DnDiscordAPI/Games/Character/Services/CharacterProgressionAdapter.cs`

- **XP (Experience Points)** - Currency of progression awarded by the DM via `DmAwardExperience`. Accumulates on the character; the server checks whether the new total crosses a level threshold after every award.

- **Inventory** - The list of items held by a character. The DM grants items via the hub (`DmGrantItem`) or the REST endpoint; players can view and use items from the hotbar. _Code:_ `back/src/DnDiscordAPI/Games/Inventory/Services/InventoryGrantAdapter.cs`

- **Wallet** - A character's currency balance, split into copper, silver, gold, and platinum. The DM adjusts it via `DmGrantGold`; the player views it in `WalletPanel`. _Code:_ `front/dndiscord-esp/src/components/WalletPanel.tsx`

---

## Discord

- **Activity** - A Discord feature that embeds a third-party web app (here: DnDiscord) as a full-screen iframe inside a voice channel. The app is loaded from Discord's `*.discordsays.com` proxy. _Code:_ `front/dndiscord-esp/src/services/discord.ts`

- **Voice channel** - The Discord channel the Activity runs inside. The client reads the `guildId` and `voiceChannelId` from the Discord Embedded App SDK context and passes them to `CreateSession`. _Code:_ `front/dndiscord-esp/src/services/discord.ts`

- **OAuth (Discord OAuth2)** - The login flow. The user is redirected to Discord's consent page, Discord returns a code, the back-end exchanges it for user data, issues a JWT, and immediately revokes the Discord access token. _Code:_ `back/src/Campaign/Controllers/` and `back/src/Auth/` module

- **Activity URL mappings** - Discord's proxy rewrites all outbound URLs (API calls, WebSocket) from the iframe to go through `*.discordsays.com`. `SignalRService` is configured with WebSocket-only transport because Discord's proxy does not support long-polling. _Code:_ `front/dndiscord-esp/src/services/signalr/SignalRService.ts`

- **CSP (Content Security Policy)** - Browser-enforced rules imposed by the Discord Activity iframe that block `eval`, `Function()`, inline blob workers, and external CDNs. Every technical constraint in the frontend (bundled fonts, `LoadAssetContainerAsync`, no blob workers) traces back to CSP. _Code:_ `front/dndiscord-esp/nginx.conf`

- **Iframe sandbox** - The combination of Discord's CSP and `X-Frame-Options` absence that lets DnDiscord embed inside Discord while restricting what code can run. nginx must omit `X-Frame-Options` and set `frame-ancestors` to `*.discordsays.com`. _Code:_ `front/dndiscord-esp/nginx.conf`

---

## Backend tech

- **REST** - Standard HTTP API (`/api/...`) consumed by the frontend via axios. Handles campaign CRUD, character management, auth, and inventory. All in-game real-time events go through SignalR instead. _Code:_ `front/dndiscord-esp/src/services/api.ts`

- **SignalR Hub** - A persistent WebSocket endpoint (built on ASP.NET Core SignalR) that the frontend connects to for real-time multiplayer events. DnDiscord has two: `GameHub` at `/hubs/game` and `MessageHub` at `/hubs/messages`. _Code:_ `back/Multiplayer/Hubs/GameHub.cs`, `back/src/DnDiscordAPI/Messages/Hubs/MessageHub.cs`

- **GameHub** - The primary SignalR hub. It owns session lifecycle, combat state transitions, and all DM tool methods. The hub layer never writes `CombatState` directly; it delegates to `CombatManager`. _Code:_ `back/Multiplayer/Hubs/GameHub.cs`

- **EF Core (Entity Framework Core)** - The ORM used to map C# model classes to PostgreSQL tables and run migrations. Migrations execute automatically at application startup. _Code:_ `back/src/Campaign/DataAccess/CampaignDbContext.cs`, `back/src/DnDiscordAPI/Games/Database/GamesDbContext.cs`

- **CampaignDbContext / GamesDbContext** - The two EF Core database contexts. `CampaignDbContext` owns campaigns, members, maps, sessions, and rolls. `GamesDbContext` owns characters, inventory, and wallet data. _Code:_ `back/src/Campaign/DataAccess/CampaignDbContext.cs`, `back/src/DnDiscordAPI/Games/Database/GamesDbContext.cs`

- **Migration drift** - The CI pipeline runs `dotnet ef migrations has-pending-model-changes` for both DbContexts to detect when a model change was committed without a matching migration file. A drifted migration would crash the application on startup. _Code:_ `back/.github/workflows/ci.yml`

- **JWT** - A signed token (7-day TTL) issued after successful Discord OAuth. The frontend stores it in localStorage and sends it as a `Bearer` header on REST calls and as a query parameter when establishing the SignalR connection.

- **Adapter** - A thin cross-module bridge class that implements an interface from one module using services from another, avoiding circular project references. Examples: `InventoryGrantAdapter` lets `GameHub` grant items without depending directly on the Games project. _Code:_ `back/src/DnDiscordAPI/Games/Inventory/Services/InventoryGrantAdapter.cs`, `back/src/DnDiscordAPI/Games/Character/Services/CharacterProgressionAdapter.cs`

- **Roll history** - A paginated log of all D20 results during a campaign session, readable by all members. Stored in `CampaignDbContext` and exposed via `GET /api/campaigns/{id}/rolls`. _Code:_ `back/src/Campaign/Controllers/RollHistoryController.cs`

- **RGPD endpoints** - GDPR-compliant endpoints: `DELETE /api/auth/me` hard-deletes all personal data and tombstones the Discord ID; `GET /api/auth/me/export` returns a JSON export of the user's data with third-party PII pseudonymised. Both are rate-limited.

---

## Frontend tech

- **SolidJS store** - A reactive data container (`createStore`) from SolidJS used to hold shared client state (session info, units, game phase, etc.). Unlike React state, `setStore({})` on a `Record` store merges rather than replaces - clearing requires explicit key deletion. _Code:_ `front/dndiscord-esp/src/game/stores/GameStateStore.ts`, `front/dndiscord-esp/src/stores/session.store.ts`

- **Signal** - A fine-grained reactive primitive from SolidJS (`createSignal`). Changing a signal re-runs only the computations that depend on it, making updates surgical rather than full re-renders.

- **BabylonJS** - The 3D rendering engine (v7.x) used for the battle map. CSP constraints require using `LoadAssetContainerAsync` and instanced templates instead of dynamic shader tricks. _Code:_ `front/dndiscord-esp/src/engine/BabylonEngine.ts`

- **AssetContainer / instanced template** - A BabylonJS pattern where a loaded glTF model lives off-screen in an `AssetContainer`; visible copies are created with `instantiateModelsToScene`. Disposing an instance never corrupts the shared template. _Code:_ `front/dndiscord-esp/src/engine/ModelLoader.ts`

- **VFX (Visual Effects)** - Particle systems and screen effects managed by `VFXManager`. Includes ambient particles (torches, dust), spell impact VFX, and post-processing overlays (bloom, vignette). Paused during map resets and resumed after load. _Code:_ `front/dndiscord-esp/src/engine/vfx/VFXManager.ts`

- **Map editor** - The in-browser tool at `/map-editor/:mapId` for placing tiles, assets, and lights on a grid to create campaign maps. Has its own standalone BabylonJS setup separate from the game engine. _Code:_ `front/dndiscord-esp/src/pages/MapEditor.tsx`

- **Hotbar** - The in-game action bar shown to players (and separately to the DM) during a session. Surfaces abilities, spells, consumables, and utilities for the active unit. _Code:_ `front/dndiscord-esp/src/components/hotbar/PlayerHotbar.tsx`, `front/dndiscord-esp/src/components/hotbar/EnemyHotbar.tsx`

- **Campaign tree canvas** - A WIP visual story-tree editor at `/campaigns/:id/manager` that lets the DM compose encounter and scene nodes for a campaign narrative. _Code:_ `front/dndiscord-esp/src/components/campaign-tree-canvas/CampagnTreeCanvas.tsx`

- **Tutorial step** - One entry in `TUTORIAL_STEPS`, each with an id, optional route navigation, an optional `data-tutorial` spotlight target, a title, and a body. The tutorial overlay walks new users through core UI surfaces. _Code:_ `front/dndiscord-esp/src/tutorial/steps.ts`

---

## Multiplayer / session

- **Session** - A live game room created by a DM for a specific campaign. Identified by a `SessionId` and a short human-readable `JoinCode`. Transitions through states: `Lobby` (waiting for players) then `InProgress`. _Code:_ `back/Multiplayer/Models/GameSession.cs`

- **Lobby** - The waiting room phase (`SessionState.Lobby`) before `StartGame` is called. Players join, select characters or quickstart templates, and wait for the DM to begin. _Code:_ `front/dndiscord-esp/src/components/LobbyScreen.tsx`

- **GameStarted** - The SignalR event broadcast by the server when the DM calls `StartGame`. It carries server-computed spawn positions for every unit and marks the session as `InProgress`. _Code:_ `back/Multiplayer/Hubs/GameHub.cs`, `front/dndiscord-esp/src/services/signalr/gameSync.ts`

- **CombatState** - The server-side in-memory record of all combat data: current phase, round number, turn order, and per-unit runtime state. All mutations go through `CombatManager`; a `SemaphoreSlim` prevents race conditions. _Code:_ `back/Multiplayer/Models/CombatState.cs`

- **Combat FSM (Finite State Machine)** - The state machine that governs combat phases: `FreeRoam` - `Preparation` - `PlayerTurn` - `EnemyTurn` - `Resolved`. Phase transitions are triggered by hub methods and enforced server-side. _Code:_ `back/Multiplayer/Services/CombatManager.cs`, `back/Multiplayer/Define.cs:CombatPhase`

- **PendingRollRequest** - A server-side record of an in-flight DM roll request sent to one or more players. Tracks which players have not yet submitted, collects results thread-safely, and marks itself complete when all players respond. _Code:_ `back/Multiplayer/Models/PendingRollRequest.cs`

- **MapSwitched** - The SignalR event broadcast when the DM calls `DmSwitchMap`. It carries the full map data so every client reloads the scene without a separate REST fetch. The client handler clears engine state then rebuilds the grid. _Code:_ `front/dndiscord-esp/src/services/signalr/mapSwitched.ts`

- **Spawn cluster** - The group of tiles reserved for a player or enemy team's starting positions, computed deterministically by `SpawnPlacementService` using an FNV-1a seed so all clients agree on placement without a broadcast. _Code:_ `back/Multiplayer/Services/SpawnPlacementService.cs`

- **TurnEnded event** - The SignalR event broadcast after a player submits `EndTurn`. It carries the next active unit id, updated HP, remaining AP, and the current phase. All clients apply it via `applyTurnEnded` to advance the turn cursor in sync. _Code:_ `front/dndiscord-esp/src/services/signalr/turnEndedLogic.ts`

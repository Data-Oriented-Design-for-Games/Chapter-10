# Chapter 10 — Multiple Enemy Types

Sample project for **Chapter 10** of [*High Performance Unity Game Development (Using data-oriented design)*](https://www.manning.com/books/high-performance-unity-game-development) by Nitzan Wilnai (Manning).

This is the survivor game from the earlier chapters. Enemies spawn in a ring around the player and walk toward it, pushing each other apart as they go. If one touches the player the game is over, and your score is the time you survived. In this version the game has more than one kind of enemy, and each kind is described by data instead of code.

## What it shows

- Each enemy type is a ScriptableObject asset (`EnemySO`) with a name, a prefab, a velocity and a radius.
- A tool-time step gives every enemy type an integer ID and bakes all the balance data into one binary file, `Assets/Resources/balance.bytes`.
- At runtime the per-type data sits in plain arrays in `Balance` (`EnemyVelocity`, `EnemyRadius`, `EnemyPrefabName`). The enemy type is the index into them.
- An enemy does not carry its own copy of that data. `GameData.EnemyType` holds one `int` per enemy, and the logic uses it to look up the velocity and radius.
- A weighted spawn table (`SpawnData` in `BalanceSO`) decides which type spawns next.
- A saved game stores the enemy type names as well as the IDs, so it can still be loaded after the IDs change.
- `Board` keeps a pool of enemy GameObjects and remembers which enemy type each pooled object is.

## What's new since [Chapter-9-Tool-Time-Data-Parsing](https://github.com/Data-Oriented-Design-for-Games/Chapter-9-Tool-Time-Data-Parsing)

- New `EnemySO.cs`, and two enemy assets in `Assets/Data/Enemies` (Zombie and Big Zombie), each with its own prefab.
- `BalanceSO` no longer has a single enemy velocity and radius. It has a `SpawnData` array instead, where each entry is an enemy and a weight.
- `BalanceParser` has a new `assignIDS` step. It then writes the spawn table and every `EnemySO` into `balance.bytes`.
- `Balance` reads the per-type arrays, and builds `EnemyNameToID` and `EnemyIDToName` for save files.
- `GameData` has a new `EnemyType` array.
- `Logic` picks a type with `getRandomEnemyTypeByWeight` when it spawns an enemy, and uses that type's velocity and radius for movement and collision.
- `GameDataIO` is now one `Save` and one `Load`. The file starts with a version number, stores `EnemyType`, and ends with the enemy type names. `Load` uses the names to fix up any type whose ID changed.
- `AssetManager.GetEnemyGameObject` takes a prefab name and picks from a list of enemy prefabs.
- `Board` no longer creates every enemy GameObject up front. `getFreeEnemyPoolIndex` reuses a free object of the right type or creates a new one, and `m_enemyToPoolIndex` maps an enemy to its pooled object.

## How the code is organized

Everything is in `Assets/Scripts`.

Data
- `GameData.cs` — the state of a running game: enemy positions and types, the alive and dead index lists, player direction, game time.
- `Balance.cs` — read-only game settings, loaded from `balance.bytes`.
- `MetaData.cs` — menu state and best time.
- `ScriptableObjects/BalanceSO.cs`, `ScriptableObjects/EnemySO.cs` — the assets you edit in the Editor. The assets themselves are in `Assets/Data`.

Logic
- `Logic.cs` — static functions that take the data and change it: spawn, move, collide, check for game over. `Logic.Tick` runs one frame and reports which enemies were added and removed.
- `GameDataIO.cs`, `MetaDataIO.cs` — save and load to binary files.

The Unity side
- `Game.cs` — the entry point. It owns the data, switches between menu states and calls `Board.Tick` every frame.
- `Board.cs` — reads input, calls `Logic.Tick`, then updates the pooled enemy GameObjects to match the data.
- `AssetManager.cs` — instantiates the player, enemy and UI prefabs.
- `MainMenuVisual.cs`, `PauseMenuVisual.cs`, `GameOverVisual.cs` — the menus.
- `Tools/GUIRef.cs`, `Tools/Singleton.cs`, `Tools/FPSCounter.cs` — small helpers.

Tools
- `BalanceParser.cs` — the editor menu command that writes `balance.bytes`.

## Running it

1. Open the project in Unity **6000.3.22f1** (Unity 6.3 LTS) or newer.
2. Open `Assets/Scenes/MainGameScene.unity` and press **Play**.
3. Click **New game**. Hold the left mouse button and drag to choose a direction. A small on-screen joystick appears where you pressed. Release to stop. The player stays in the middle of the screen and the enemies move around it.
4. The `| |` button pauses the game and saves it. When a saved game exists, **Continue** on the main menu loads it. Press **S** to save a screenshot.
5. To change the game data, edit `Assets/Data/Balance.asset` or the assets in `Assets/Data/Enemies`, then run **DOD > Balance > Parse Local** from the menu bar. This rewrites `Assets/Resources/balance.bytes`, which is the file the game reads.

`Board.handleInput` uses the mouse in the Editor and the first touch on a device.

To add an enemy type, create an `EnemySO` in `Assets/Data/Enemies` (**Assets > Create > DOD > EnemySO**), add its prefab to the `m_enemyPrefabs` list on the `AssetManager` component in the scene, add it to `SpawnData` in `Balance.asset`, and run **Parse Local** again.

## More samples

All sample projects for the book: https://github.com/Data-Oriented-Design-for-Games

Next: [Chapter-11](https://github.com/Data-Oriented-Design-for-Games/Chapter-11)

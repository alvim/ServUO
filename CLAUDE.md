# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ServUO is an Ultima Online Server Emulator written in C# targeting .NET Framework 4.8. It's a community-driven project that emulates the classic MMORPG Ultima Online, handling networking, game logic, world state, and content scripting.

## Build System

### Windows
- Debug build: `_windebug.bat` - Runs `dotnet build -c Debug` and launches with `-debug` flag
- Release build: `_winrelease.bat` - Runs `dotnet build -c Release` and launches normally

### Linux/macOS
- Debug build: `make debug`
- Release build: `make` or `make release`
- Build only: `make build`
- Clean: `make clean`

All builds output to the repository root directory. The executable is `ServUO.exe` (run via mono on Linux/macOS).

## Solution Structure

The solution contains three projects:

1. **Ultima** (`Ultima/Ultima.csproj`) - Low-level library for reading Ultima Online client data files
2. **Server** (`Server/Server.csproj`) - Core server engine (executable assembly)
3. **Scripts** (`Scripts/Scripts.csproj`) - Game content and logic (library assembly, depends on Server)

Build configuration is x64 only with Debug/Release modes.

## Core Architecture

### Server Project (`Server/`)

The Server project contains the core emulator engine:

- **Main.cs** - Entry point (`Server.Core` class with startup logic)
- **World.cs** - Central world state manager, handles all entities (Mobiles, Items, SaveData), serialization to `Saves/` directory
- **Item.cs** / **Mobile.cs** - Base classes for all game objects and creatures
- **Network/** - Client networking layer (PacketHandlers, NetState, Listener, ByteQueue, Compression)
- **Config.cs** - Configuration system that loads `*.cfg` files from `Config/` directory
- **EventSink.cs** - Central event system for hooking into server events
- **Serialization.cs** - Binary serialization infrastructure for world saves
- **Timer.cs** / **Timers/** - Game tick and scheduling system
- **Customs Framework/** - Modular framework for custom functionality (BaseCore, BaseModule, BaseService, CustomSerial)

### Scripts Project (`Scripts/`)

Game content is entirely scripted in C# and organized by type:

- **Mobiles/** - NPCs, monsters, and AI
- **Items/** - Equipment, consumables, containers, etc.
- **Spells/** - All spell implementations organized by spell circle
- **Skills/** - Skill system implementations
- **Services/** - Major game systems (BulkOrders, ChampionSystem, Factions, VvV, Crafting, etc.)
- **Commands/** - GM/Admin commands
- **Gumps/** - UI dialog implementations
- **Regions/** - Area-specific logic (towns, dungeons, etc.)
- **Quests/** - Quest system implementations
- **Misc/** - Utility scripts (AutoSave, AutoRestart, CharacterCreation, CurrentExpansion, etc.)

### Configuration System

All configuration files live in `Config/` and use a custom format:

- Lines starting with `#` are descriptions
- `Key=Value` syntax
- `@Key=Value` forces default value and suppresses warnings
- Empty keys result in null/default values
- Blank lines terminate option blocks

Key configuration files:
- `Server.cfg` - Server name, IP, port
- `DataPath.cfg` - Path to UO client data files
- `Expansion.cfg` - Enabled UO expansions
- `AutoSave.cfg` / `AutoRestart.cfg` - Server maintenance

Start by reading `Config/README.md` when setting up a server.

## World State and Persistence

World state is saved to `Saves/` directory:

- `Saves/Mobiles/` - All mobile entities (Mobiles.idx, Mobiles.tdb, Mobiles.bin)
- `Saves/Items/` - All item entities (Items.idx, Items.tdb, Items.bin)
- `Saves/Guilds/` - Guild data (Guilds.idx, Guilds.bin)
- `Saves/Customs/` - Custom SaveData entities (SaveData.idx, SaveData.tdb, SaveData.bin)

The World class manages serialization/deserialization during server startup and shutdown.

## Key Patterns

### Entity Lifecycle
- All entities inherit from Item or Mobile
- Entities use a Serial-based system for unique identification
- World state is managed through World.Items and World.Mobiles dictionaries
- Serialization/deserialization via overridden Serialize/Deserialize methods

### Event-Driven Architecture
- EventSink provides hooks for major server events (Login, Speech, Movement, etc.)
- Scripts register event handlers during initialization
- Customs Framework extends this with BaseCore/BaseModule pattern

### Networking
- Incoming packets handled by PacketHandlers (registered in PacketHandlers.cs)
- Outgoing packets defined in Packets.cs
- NetState represents a client connection
- ByteQueue manages buffering, Compression handles packet compression

### Timers
- Timer base class for scheduled/recurring tasks
- DelayCall for one-time delayed execution
- Timers run on dedicated timer thread

## Common Development Tasks

### Adding New Content
- Items: Create class inheriting from Item in `Scripts/Items/`
- Mobiles: Create class inheriting from Mobile (or BaseCreature) in `Scripts/Mobiles/`
- Override Serialize/Deserialize for any custom state
- Register in appropriate initialization hooks if needed

### Modifying Server Behavior
- Hook into EventSink for broad behavioral changes
- Use Customs Framework BaseCore/BaseModule for modular custom systems
- Modify `Scripts/Misc/` files for startup/core behavior changes

### Configuration Changes
- Edit appropriate `*.cfg` file in `Config/`
- Add new config properties by decorating them with [ConfigProperty] attribute in relevant code
- Config system automatically loads and parses on startup

## Platform Notes

### Linux Dependencies (from README)
- **Ubuntu/Debian**: `sudo apt-get install zlib1g mono-complete dotnet-sdk-10.0 dotnet-runtime-10.0`
- **Arch**: `sudo pacman -S make mono dotnet-sdk dotnet-runtime`

### Compiler Defines
- `NEWTIMERS` - Always defined (modern timer implementation)
- `ServUO` - Always defined
- `DEBUG` - Set in Debug builds
- `TRACE` - Always defined

## Data Files

The `Data/` directory contains static game data:
- Item/mobile definitions (`items.cfg`, `mobiles.cfg`)
- Body table mappings (`bodyTable.cfg`)
- Spawn locations, decoration, and map data
- NPC names (`names.xml`)
- Component definitions

This data is loaded at startup and used to populate the game world.

## Testing and Debugging

- Use Debug builds (`_windebug.bat` or `make debug`) for development
- Attach debugger to ServUO.exe process
- Console commands available via `Scripts/Misc/ConsoleCommands.cs`
- In-game commands for GMs defined in `Scripts/Commands/`

---

## Development Journey - Project Context

### Current Achievement (2025-12-11)
- ServUO server running successfully on Kali Linux (WSL2)
- ClassicUO client (Windows) connecting to ServUO server (WSL) - cross-platform setup working!

### Development Environment
- **Server**: ServUO on Kali Linux (WSL2) at /home/alvim/repos/ServUO
- **Client**: ClassicUO running on Windows host
- **Platform**: Linux-Windows hybrid setup (WSL2 networking bridge working)
- **Repository**: Git branch 'first-run', clean status

### Project Goals
- Long-term hobby project to learn ServUO (server) and ClassicUO (client) development
- Create custom variations and modifications of both
- Learning approach: Interactive, ludic (playful), intuitive, day-by-day
- This is the beginning - documenting from day one

### Tech Stack Understanding
- ServUO: C# (.NET Framework 4.8), Ultima Online Server Emulator
- Core projects: Ultima library, Server engine, Scripts (game content)
- Build: `make debug` or `make release` on Linux
- Architecture: Entity-based (Items/Mobiles), event-driven, serialization-based persistence

### Learning Philosophy
- Take time, enjoy the process
- Build knowledge incrementally
- Experiment and play with the codebase
- Document discoveries along the way
- Claude Code as development companion throughout the journey

### Next Steps (When Ready)
- Explore the codebase gradually
- Understand core systems (World, Items, Mobiles, Network)
- Try small modifications
- Learn the Scripts architecture
- Eventually tackle ClassicUO client development

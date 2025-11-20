# TomeNET Development Reference

## Project Overview
TomeNET is a multiplayer online roguelike game based on Angband with a client-server architecture using UDP networking with a reliable transmission layer built on top.

## Core Architecture

### Directory Structure
- `src/server/` - Server code (game logic, networking, world simulation)
- `src/client/` - Client code (UI, input handling, graphics/sound)
- `src/common/` - Shared utilities (networking, data structures, random)
- `src/account/` - Account management utilities
- `lib/game/` - Game data files (*_info.txt format)
- `lib/scpt/` - Lua script files for game logic
- `lib/config/` - Configuration files (tomenet.cfg)
- `lib/data/` - Runtime server data (logs, scores, etc.)
- `lib/save/` - Player save files and server state

### Key Binaries
- `tomenet` - Client executable
- `tomenet.server` - Server executable  
- `accedit` - Account editor utility

### Build System
- **Main makefile**: `src/makefile` - Primary build configuration
- **Platform variants**:
  - `makefile.mingw` - Windows cross-compilation
  - `makefile.gcu` - ncurses terminal client
  - `makefile.osx` - macOS specific
- **Client variants**:
  - X11 GUI: `-DUSE_X11` flag
  - ncurses terminal: `-DUSE_GCU` flag
  - Windows: `-DWIN32` flag
- **Features**:
  - SDL sound: `-DSOUND_SDL` with SDL2_mixer dependency
  - Lua integration: Built-in with embedded interpreter and tolua bindings
  - Preprocessor support: cpp for conditional compilation in Lua files

### Key Systems

#### Networking
- **Main server**: `src/server/main.c` - Entry point with `play_game()` game loop
- **Network layer**: `src/server/nserver.c` - Connection states and packet handling
- **Scheduler**: Turn-based gameplay using `sched()` function

#### Scripting
- **Lua integration**: `src/server/script.c` with `exec_lua()` and `pern_dofile()`
- **Script files**: `lib/scpt/` directory contains game logic scripts
- **Preprocessing**: Lua files support cpp conditional compilation

#### Data Files
- **Format**: Structured text files (*_info.txt)
- **Key files**:
  - `r_info.txt` - Monster definitions
  - `k_info.txt` - Object/item definitions
  - `a_info.txt` - Artifact definitions
  - `d_info.txt` - Dungeon definitions
  - `t_info.txt` - Town/terrain definitions

#### Configuration
- **Main config**: `lib/config/tomenet.cfg`
- **Contains**: Game settings, network options, admin controls
- **Additional**: banlist.txt, badnames.txt, server_portals.cfg

## Development Practices

### Code Style
- Keep functions short and descriptive
- No excessive commenting - code should be self-explanatory
- No defensive programming or unnecessary try/catch blocks
- No function docstrings
- Avoid list comprehensions
- Use common sense with newlines - don't add extras everywhere
- One import per line, reorder logically when editing Python files

### Git Workflow
- `/ca` command: Add all new/changed files and commit with generated message
- `/cb` command: Same as `/ca` but message is just "backup"
- Files ending in `~` are ignored (backup files)

### Environment Setup
- Use appropriate venv (typically `.venv`) for Python code
- For editing system files (`/etc`), use sudo shell commands instead of direct file editing
- Don't use exclamation marks in terminal commands

### Build Notes
- Post-commit hook copies binaries from `src/` to root directory
- Use `make install` or manually copy binaries after building
- Object files (*.o) are automatically ignored

## File Patterns to Ignore
- `*.o` - Object files
- `*~` - Editor backup files
- `*.tmp`, `*.dna`, `*.xhtml` - Temporary files
- `lib/data/*` - Runtime server data
- `lib/save/*` - Save files
- Generated tolua and preproc files

## Common Development Tasks

### Building
```bash
make                    # Build all targets
make tomenet           # Build client only
make tomenet.server    # Build server only
make clean             # Clean build artifacts
```

### Running
- `./runserv` - Simple server startup script
- `./runserv3` - More complex server startup with additional options
- `./tomenet` - Client executable

### Account Management
- `./accedit` - Account editor utility
- Account data stored in `tomenet.acc`

## Network Architecture

### Server (`src/server/nserver.c`)
- **Protocol**: UDP-based with custom reliability layer piggybacked on unreliable packets
- **Connection States**:
  1. Initial socket setup
  2. Client authentication (name/params)
  3. Server config transmission
  4. Ready-but-not-playing state
  5. Output drain states (ACK waiting)
  6. Actively playing
- **Reliability**: Client ACKs byte position in reliable stream, server retransmits on timeout
- **RTT Tracking**: Adaptive retransmit timeout based on round-trip time measurements
- **Key Functions**: `Net_input()`, `Net_output()`, `Send_reliable()`, `Receive_ack()`

### Client (`src/client/nclient.c`)
- Packet dispatch via `receive_tbl[]` lookup table
- Socket buffer with rollback support for incomplete packets
- Handles map updates, messages, player stats, etc.

### Game Loop Integration
- Server: `dungeon()` calls `Net_input()` at start, `Net_output()` at end
- Client: Main loop polls `Net_input()` via `SocketReadable()`
- Keepalive packets maintain UDP routing table priority

## Lua Integration

### Initialization (`src/server/script.c`)
- **Lua State**: Single global `L` for all server scripts
- **Libraries**: Base, math, string, io, debug + custom bitlib
- **APIs**: tolua-generated bindings (util, player, spells, z_pack)
- **Startup**: `init_lua()` loads `init.lua`, then initializes schools/spells from Lua

### Execution Model
- `pern_dofile(Ind, "file.lua")` - Load and execute Lua file
- `exec_lua(Ind, "code")` - Execute string, return numeric result
- `string_exec_lua(Ind, "code")` - Execute string, return string result
- `Ind` parameter: Player index (0 = server context, 1+ = specific player)
- Sets `Ind` and `player` globals before each execution

### Hooks & Callbacks
- `second_handler()` - Called every server second (from `dungeon()` loop)
- Spell casting: Lua defines spell effects, C code invokes them
- Quest triggers: Lua scripts handle quest logic
- Custom events: Lua can be invoked from C at any point

### Lua Files (`lib/scpt/`)
- **Core**: `init.lua`, `module.lua`, `xml.lua`
- **Spells**: `s_*.lua` (fire, water, air, earth, mind, nature, etc.)
- **Prayers**: `p_*.lua` (offense, defense, support, curing)
- **Occult**: `o_*.lua` (shadow, spirit, hereticism, unlife)
- **Character**: `classes.lua`, `races.lua`, `traits.lua`
- **Systems**: `spells.lua`, `powers.lua`, `quests.lua`, `player.lua`

## Data File Format

### Common Structure
All `*_info.txt` files use line-prefix format:
- `V:` - Version stamp (required first line)
- `N:idx:name` - Name/index (idx often ignored, auto-numbered)
- `I:` - Info (stats vary by file type)
- `W:` - Weight/depth/rarity
- `F:` - Flags (pipe-separated: `FLAG1 | FLAG2`)
- `D:` - Description text (can span multiple lines)

### Parsing (`src/server/init2.c`)
1. Allocate header + info array in memory
2. Parse line-by-line with state machine
3. Optionally cache as binary `.raw` file in `lib/data/`
4. Check modification time to decide whether to re-parse

### Major Files
- `r_info.txt` (20K+ lines) - Monster races: HP, AC, attacks, spells, AI flags
- `k_info.txt` (8K+ lines) - Object kinds: tval/sval, weight, damage, flags
- `a_info.txt` - Artifacts: Unique items with special powers
- `e_info.txt` - Ego items: Item modifiers (of Slaying, of Resist, etc.)
- `d_info.txt` - Dungeons: Depth ranges, monster/object tables
- `f_info.txt` - Features: Terrain types (walls, floors, doors)
- `t_*.txt` - Town layouts: ASCII maps with feature codes

## Quick Index

### Find specific functionality:
- **Player commands**: `src/server/cmd*.c` (cmd1.c = movement, cmd2.c = items, etc.)
- **Combat**: `src/server/melee*.c`, `attack.c`
- **Spells**: `src/server/spells*.c`
- **Monsters**: `src/server/monster*.c`, `mon-ai.c`
- **Generation**: `src/server/generate.c`, `wild.c`
- **Network**: `src/server/nserver.c`, `src/client/nclient.c`
- **UI**: `src/client/z-term.c`, `main-*.c`
- **Game loop**: `src/server/dungeon.c` (line 10343)
- **Client loop**: `src/client/c-init.c` (line 3133)

### Important globals:
- `Players[]` - Player array (1-indexed! 0 = server context)
- `NumPlayers` - Current connected players
- `turn` - Game turn counter (s32b)
- `m_list[]`, `o_list[]` - Monster/object arrays
- `r_info[]`, `k_info[]`, `a_info[]` - Parsed data tables
- `L` - Lua state
- `cfg` - Server config structure

## Additional Resources

For a comprehensive index with detailed system explanations, see `.cursor_index.md`.

This reference should be updated as new information about the codebase structure and development practices is discovered.

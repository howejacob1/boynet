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
- UDP-based with reliability layer
- Client-server model
- Connection state management in nserver.c
- Packet-based communication protocol

## Lua Integration
- Embedded Lua interpreter
- tolua bindings for C<->Lua interface
- Script files in `lib/scpt/`
- Preprocessor support for conditional compilation
- Main script functions: `exec_lua()`, `pern_dofile()`

This reference should be updated as new information about the codebase structure and development practices is discovered.

# User Configuration Directory

This directory contains both **default configuration files** (tracked by git) and **user-generated files** (ignored by git).

## How It Works

### Default Configs (Tracked by Git)
These files are shipped with the game and will be updated when you pull changes:
- `global.opt`, `global.prf` - Global settings
- `font-*.prf` - Font configurations
- `pref-*.prf` - Platform-specific preferences
- `graf-*.prf` - Graphics configurations  
- `arcade-*.prf` - Arcade mode settings
- `graphics-*.prf` - Graphics tile mappings
- `linux_*.prf` - Linux-specific configs
- `options.prf` - Options reference

### User Files (Ignored by Git)
These are automatically ignored and won't be tracked or overwritten:
- `<CharacterName>.opt` - Character-specific options
- `<CharacterName>.prf` - Character-specific preferences
- `<CharacterName>.dna` - Character DNA/macros
- `<CharacterName>-death_*.txt` - Death dumps
- `chathist-*.tmp` - Chat history
- `tomenet-messages_*.txt` - Message logs
- `screenshot_*.xhtml` - Screenshots

## Safe to Update

When you `git pull` updates:
- ✅ Default configs will update to new versions
- ✅ Your character configs are safe and won't be touched
- ✅ Your personal settings are preserved

If you've modified a default config and want to keep your changes, copy it to a character-specific name (e.g., `global.prf` → `YourCharacter.prf`).


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Signals Plus signal system mod for NIMBY Rails. Implements 5-aspect signaling (Idle, Pass, Stop, Caution, Drive on Sight) with automatic speed enforcement. Includes three built-in signal shapes (Pill, Circle, Triangle).

## Development Workflow

1. Edit `src/signals-plus.nimbyscript` directly
2. NIMBY Rails hot-reloads scripts on save
3. Test in-game by placing signals and running trains

No build step required - the game interprets NimbyScript directly.

## Key Documentation

- **NimbyScript Language**: https://wiki.nimbyrails.com/NimbyScript
- **Modding Guide**: https://steamcommunity.com/sharedfiles/filedetails/?id=2268014666
- **Detailed NimbyScript Reference**: See NimbyScript wiki

## NimbyScript Critical Rules

These language quirks cause common errors:

1. **Integer division**: Use `zdiv(a, b)` and `zmod(a, b)` - the `/` and `%` operators only work for floats
2. **No method chaining**: `obj.method1().method2()` won't compile - use intermediate variables
3. **Explicit return required**: Functions must use `return` statement
4. **Pointer validation**: Use `if let x &= pointer { }` or `let x &= pointer else { return; }`
5. **Private structs deleted end-of-frame**: Must attach to game objects via `sc.queue_attach()`
6. **No private methods on pub structs**: Inline the logic instead
7. **Complex types in function params**: Types like `&Vec<ID<Signal>>` only work as struct fields

## Architecture

### Script Structure (`src/signals-plus.nimbyscript`)

- **SignalPhase enum**: Idle, Pass, Stop, Caution, DriveOnSight
- **SignalPlusSignal pub struct**: Extends Signal, player-configurable `signals_ahead` list
- **SignalPlusStation pub struct**: Extends Station, `approach_signals` for caution display
- **SignalState private struct**: Runtime state attached to signals

### Event Handlers

| Handler | Purpose |
|---------|---------|
| `event_signal_lookahead` | Called up to 15km ahead, sets speed limits and signal state |
| `event_signal_pass_by` | Resets state when train passes |
| `event_signal_texture_state` | Returns texture index based on signal state and blink cycle |

### Signal Shapes

Three texture sets are defined in `mod.txt`, each as a separate `[SignalTextures]` + `[SignalTemplate]` pair:
- **Pill** (`SIGNALS_PLUS_TEXTURES`) - Rounded rectangle
- **Circle** (`SIGNALS_PLUS_TEXTURES_CIRCLE`) - Circular
- **Triangle** (`SIGNALS_PLUS_TEXTURES_TRIANGLE`) - Triangular

### Texture Mapping

Each shape has 4 textures indexed in order:
- 0: `color=none` (off/idle)
- 1: `color=green` (pass)
- 2: `color=red` (stop)
- 3: `color=amber` (caution / drive on sight)

Texture naming convention: `textures/color={none,green,red,amber}-shape={pill,circle,triangle}.png`

Textures must be 64x64 pixels, 32-bit RGBA PNG.

## Debugging

Use `log("message", var1, var2)` - enable logging in game UI (has CPU cost).

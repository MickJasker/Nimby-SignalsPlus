# AGENTS.md - NimbyScript Development Guide

## Project Overview

This project implements ATB (Automatic Train Protection) signals for NIMBY Rails using NimbyScript. The goal is to create realistic signal behavior that enforces speed limits and stops based on signal state and train dynamics.

## Documentation Reference

- **NimbyScript Language**: https://wiki.nimbyrails.com/NimbyScript
- **Modding Guide**: https://steamcommunity.com/sharedfiles/filedetails/?id=2268014666

Always consult the wiki for the most up-to-date API reference and language features.

---

## Mod Configuration (mod.txt)

The `mod.txt` file configures the mod using INI syntax. Required and optional sections:

### [ModMeta] - Required

```ini
[ModMeta]
schema=1
name=Mod Name
author=Author Name
desc=Description of the mod
version=1.0.0
```

### [Script] - For NimbyScript mods

```ini
[Script]
id=UNIQUE_SCRIPT_ID
name=Display Name
source=src/script.nimbyscript
```

### [SignalTextures] - For custom signal visuals

```ini
[SignalTextures]
id=UNIQUE_TEXTURE_ID
name_loc=localization_key
name_en=English Name
state=textures/state0.png
state=textures/state1.png
state=textures/state2.png
```

- Textures are listed in order matching `event_signal_texture_state` return values
- Textures must be 64x64 pixels, 32-bit RGBA PNG

### [SignalTemplate] - For custom signal types

```ini
[SignalTemplate]
name_en=English Name
name_loc=localization_key
kind=path
textures=TEXTURE_ID_REFERENCE
extend=PubStructName
```

- `kind`: `path`, `balise`, `oneway`, `noway`, `marker`, or `platform_stop`
- `extend`: References the `pub struct` name from your script
- `textures`: References a `[SignalTextures]` id

### Naming Conventions

- IDs must be globally unique across all mods
- Use only Latin characters, digits, and underscores
- Use UPPERCASE for IDs by convention

---

## NimbyScript Language Quick Reference

### Script Structure

Every NimbyScript file must start with the meta declaration:

```nimbyscript
script meta {
  lang: nimbyscript.v1,
  api: nimbyrails.v1,
}
```

### Data Types

| Type | Description |
|------|-------------|
| `i8`, `i16`, `i32`, `i64` | Signed integers |
| `u8`, `u16`, `u32`, `u64` | Unsigned integers |
| `f32`, `f64` | Floating point |
| `bool` | Boolean |
| `&Type` | Reference (permanent binding) |
| `*Type` | Pointer (requires validity check) |

### Variable Declaration

```nimbyscript
let x = 1;              // immutable, type inferred as i64
let y: f64 = 1.0;       // explicit type
let z mut= 1;           // mutable variable
let r &= variable;      // reference binding (permanent alias)
```

### Structs

**Public structs (`pub struct`)**: Exposed in game UI, player-owned, support schema evolution
```nimbyscript
pub struct MySignal extend Signal {
  // fields visible in UI
}
```

**Private structs**: Script-owned simulation state, support dynamic allocation
```nimbyscript
struct MyState {
  value: i64,
  signal_id: ID<Signal>,
}
```

### Enums

```nimbyscript
pub enum SignalState {
  Idle,
  Pass,
  Stop,
}

let state = SignalState::Idle;
```

### Control Flow

```nimbyscript
// if/else
if condition {
  // ...
} else if other {
  // ...
} else {
  // ...
}

// Pointer validation with let else
let obj &= db.view<Type>(ref) else { return; }

// Pointer validation with if let
if let ref &= pointer {
  // pointer is valid
}
```

### Functions

```nimbyscript
fn helper_function(param: i64): f64 {
  return param as f64;
}

pub fn StructName::method_name(self: &StructName, ctx: &EventCtx): Type {
  // extension method
}
```

---

## Critical Language Rules

### Integer division and modulo require safe functions
The `/` and `%` operators work for `f64` but **not for integers**. Use safe alternatives for integers:
```nimbyscript
let result = zdiv(a, b);  // integer division, returns 0 if b is 0
let remainder = zmod(a, b);  // integer modulo, returns 0 if b is 0

// For floats, regular division works:
let f_result = speed * speed / (2.0 * max_braking);
```

### Method chaining is limited to one level
```nimbyscript
// BAD - will not compile
let x = obj.method1().method2();

// GOOD - use intermediate variables
let temp = obj.method1();
let x = temp.method2();
```

### Variables cannot be reassigned (references bind permanently)
```nimbyscript
// References are permanent bindings
let r &= variable;  // r is forever bound to variable
```

### Explicit return required
```nimbyscript
fn get_value(): i64 {
  return 42;  // explicit return required
}
```

### Type conversion
Use `as_xxx()` functions for type conversion:
```nimbyscript
let f: f64 = 3.14;
let i = as_i64(f);
```

---

## Signal Extension API

### Available Event Handlers

| Event | Purpose | When Called |
|-------|---------|-------------|
| `event_signal_check` | Path validation, can stop trains | During pathfinding |
| `event_signal_lookahead` | Speed limit enforcement | 15km ahead of train |
| `event_signal_pass_by` | Actions when train passes | High privilege |
| `event_signal_texture_state` | Visual display state | For rendering |
| `event_signal_marker_reserved` | Track reservation reports | On reservation |

### event_signal_lookahead Signature

```nimbyscript
pub fn MySignal::event_signal_lookahead(
  self: &MySignal,
  ctx: &EventCtx,
  train: &Train,
  motion: &Motion,
  signal: &Signal,
  train_distance: f64,
  check: SignalCheck,
  sc: &mut SimpleSimController,
  result: &mut SignalLookaheadResult
) {
  // Set speed limits via result.max_speed
}
```

### event_signal_pass_by Signature

```nimbyscript
pub fn MySignal::event_signal_pass_by(
  self: &MySignal,
  ctx: &EventCtx,
  train: &Train,
  motion: &Motion,
  signal: &Signal,
  sc: &mut SimController
) {
  // Actions when train passes signal
}
```

### event_signal_texture_state Signature

```nimbyscript
pub fn MySignal::event_signal_texture_state(
  self: &MySignal,
  db: &DB,
  signal: &Signal,
  clock_us: i64
): i64 {
  // Return texture index (0, 1, 2, etc.)
  return 0;
}
```

---

## SimController API

Queue commands for asynchronous execution:

```nimbyscript
sc.queue_attach(object, struct);     // Attach private struct to game object
sc.queue_erase(struct);              // Delete attached struct
sc.queue_task(struct);               // Execute struct's task_run() method
sc.queue_task_at(struct, deadline);  // Schedule task at specific time
```

---

## Database Access (DB)

Read-only game state:

```nimbyscript
// View a script struct attached to an object
let state = db.view<MyState>(signal.id);

// View game object by ID
let train = db.view<Train>(train_id);
```

---

## Train Motion Properties

Access via `motion` parameter:

```nimbyscript
let speed = motion.presence.get().speed;
let dynamics = motion.dynamics;
let max_speed = dynamics.max_speed;
let max_braking = dynamics.max_regular_braking;
let max_emergency = dynamics.max_emergency_braking;
let mass = dynamics.mass;
let length = dynamics.length;
```

---

## Common Patterns

### Calculating Stopping Distance

```nimbyscript
let speed = motion.presence.get().speed;
let max_braking = motion.dynamics.max_regular_braking;
let stopping_distance = speed * speed / (2.0 * max_braking);
```

### Attaching State to Signal

```nimbyscript
let task = MyState::new(initial_value, signal.id);
sc.queue_attach(signal, task);
```

### Reading Attached State

```nimbyscript
let state = db.view<MyState>(signal.id);
if let s &= state {
  // state exists and is valid
}
```

---

## Debugging

Use `log()` for debug output (has significant CPU cost, disabled by default):

```nimbyscript
log("Signal {}: distance={}, speed={}", signal.id, distance, speed);
```

Enable logging manually in the game UI.

---

## Common Mistakes to Avoid

1. **Using `/` or `%` on integers** - Use `zdiv()` and `zmod()` for integers (floats can use `/`)
2. **Chaining multiple method calls** - Use intermediate variables
3. **Forgetting explicit `return`** - Always use `return` statement
4. **Comparing pointers directly** - Use `if let` or `let else` for validation
5. **Modifying pub struct schema incorrectly** - Only add/reorder fields, never rename or change types
6. **Not attaching private structs** - They're deleted end-of-frame unless attached to game objects
7. **Defining private methods on pub structs** - Private functions cannot be methods on pub structs; inline the logic instead
8. **Using complex types in standalone functions** - Types like `&Vec<ID<Signal>>` can only be used as struct fields, not function parameters

---

## Project-Specific Notes

This project implements ATB (Automatic Train Braking) signals.

### Script Structure

- **AtbSignalState enum**: `Idle`, `Pass`, `Stop`
- **AtbSignal pub struct**: Extends Signal, contains `signals_ahead` list
- **AtbSignalTask private struct**: Runtime state with `state` and `station_stop_ahead`

### Signal Behavior

The signal displays different textures based on state:

| Texture Index | State | Condition |
|---------------|-------|-----------|
| 0 | Idle | Train > 1km away or outside braking range |
| 1 | Pass | Signal clear, no stops ahead |
| 2 | Stop | Signal shows stop |
| 3 | Caution | Pass but stop signal ahead OR station stop within braking distance |

### Speed Limits

When approaching a stop signal:
- Within 25m: max 10 km/h
- Within 100m: max 40 km/h

### Mod Files

- `src/atb.nimbyscript` - Signal logic
- `mod.txt` - Mod configuration
- `textures/` - Signal state textures (idle, pass, stop, caution)

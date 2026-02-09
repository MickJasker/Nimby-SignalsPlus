# Signals Plus for NIMBY Rails

Signals Plus signal system for NIMBY Rails. Provides realistic signal aspects that warn trains of upcoming stops and enforce speed limits.

## Features

- **4-aspect signaling**: Idle, Pass, Stop, and Caution states
- **Automatic caution detection**: Shows caution when:
  - A signal ahead displays Stop
  - A scheduled station stop is within braking distance
- **Speed enforcement**: Automatically limits train speed when approaching stop signals
- **Configurable signal chains**: Link signals together via the `signals_ahead` field

## Installation

1. Download or clone this repository
2. Copy the mod folder to your NIMBY Rails mods directory:
   - Windows: `%APPDATA%\NIMBY Rails\mods\`
   - macOS: `~/Library/Application Support/NIMBY Rails/mods/`
   - Linux: `~/.local/share/NIMBY Rails/mods/`
3. Enable the mod in-game via the Mods menu

## Usage

### Placing Signals Plus Signals

1. Open the signal placement tool in NIMBY Rails
2. Select "Signals Plus Signal" from the signal types
3. Place signals along your track

### Configuring Signal Chains

To enable caution aspects, you need to link signals together:

1. Select a Signals Plus Signal
2. In the signal properties, find the `signals_ahead` field
3. Add the IDs of signals that are ahead on the track
4. When any linked signal shows Stop, this signal will show Caution

### Signal Aspects

| Aspect | Meaning |
|--------|---------|
| Idle (dark) | No train approaching within 1km |
| Pass (green) | Clear to proceed at full speed |
| Caution (yellow) | Prepare to stop - reduce speed |
| Stop (red) | Stop at this signal |

### Speed Limits

When approaching a Stop signal, Signals Plus enforces:
- **Within 100m**: Maximum 40 km/h
- **Within 25m**: Maximum 10 km/h

## Development

### Prerequisites

- NIMBY Rails with modding support
- Text editor for NimbyScript development

### Project Structure

```
nimby-atb-signals/
├── mod.txt                 # Mod configuration
├── src/
│   ├── signals-plus.nimbyscript    # Signal logic
│   └── AGENTS.md          # AI development guide
├── textures/
│   ├── idle.png           # Idle state texture
│   ├── pass.png           # Pass state texture
│   ├── stop.png           # Stop state texture
│   └── caution.png        # Caution state texture
└── README.md
```

### Key Components

**SignalPhase** - Enum defining signal states:
- `Idle` - No active display
- `Pass` - Clear to proceed
- `Stop` - Train must stop
- `Caution` - Prepare to stop

**SignalPlusSignal** - Public struct extending Signal:
- `signals_ahead` - List of signal IDs to check for caution state

**SignalState** - Private runtime state:
- `state` - Current signal state
- `is_approaching_station` - Whether approaching a configured station

### Event Handlers

| Handler | Purpose |
|---------|---------|
| `event_signal_lookahead` | Called up to 15km ahead, sets speed limits and state |
| `event_signal_pass_by` | Resets signal to Idle when train passes |
| `event_signal_texture_state` | Returns texture index for current state |

### Building & Testing

1. Edit files in the `src/` directory
2. Save changes - NIMBY Rails hot-reloads scripts
3. Test in-game by placing signals and running trains

### Documentation

- [NimbyScript Language Reference](https://wiki.nimbyrails.com/NimbyScript)
- [NIMBY Rails Modding Guide](https://steamcommunity.com/sharedfiles/filedetails/?id=2268014666)
- [AGENTS.md](src/AGENTS.md) - Detailed development guide for AI assistants

## Texture Requirements

Signal textures must be:
- 64x64 pixels
- PNG format
- 32-bit RGBA color (8 bits per channel)

## License

MIT License - Feel free to use, modify, and distribute.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test in-game
5. Submit a pull request

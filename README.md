# Signals Plus for NIMBY Rails

A realistic 4-aspect signal system that brings intelligent signaling to your railway. Signals automatically detect train positions and upcoming stops to display the appropriate aspect.

## Features

**4-Aspect Signaling**
- **Idle** - No train approaching (signal dark or dim)
- **Pass** - Clear to proceed at full speed
- **Caution** - Prepare to stop ahead
- **Stop** - Train must halt at this signal

**Intelligent Caution Detection**
Signals automatically show caution when:
- A linked signal ahead displays Stop
- A train is approaching a scheduled station stop

**Automatic Speed Enforcement**
Trains are automatically slowed when approaching a Stop signal to ensure safe braking.

**Blinking Support**
Each signal state can be configured to blink, cycling between the state texture and an "off" texture. Perfect for creating realistic flashing signals.

## Works With Any Signal Model

This script is designed to work with **any signal that has multiple textures**. You are not limited to the included textures - use it with your own custom signal models or other workshop assets.

### Configuring Custom Texture IDs

Each signal instance can be individually configured with custom texture IDs in the signal properties:

| Setting | Description | Default |
|---------|-------------|---------|
| **Off Texture ID** | Texture shown during blink "off" phase | 0 |
| **Idle Texture ID** | Texture when no train is approaching | 0 |
| **Pass Texture ID** | Texture when clear to proceed | 1 |
| **Stop Texture ID** | Texture when train must stop | 2 |
| **Caution Texture ID** | Texture when approaching stop/station | 3 |

Simply set these IDs to match the texture indices of your signal model. This allows one script to control signals with completely different texture layouts.

### Blinking Options

For each state, you can enable blinking:
- **Blink on Idle** - Flash when idle
- **Blink on Pass** - Flash when showing pass
- **Blink on Stop** - Flash when showing stop
- **Blink on Caution** - Flash when showing caution

When blinking is enabled, the signal alternates between the state's texture and the Off texture.

> Make sure that your signal model has an off texture to make blinking work properly.

## Setup Guide

### Linking Signals Together

To enable caution aspects based on signals ahead:
1. Select a Signals Plus Signal
2. Find the **Signals Ahead** field in signal properties
3. Add the IDs of signals further down the track
4. When a linked signal shows Stop, this signal will show Caution

### Station Approach Signals

To show caution when trains approach scheduled stops:
1. Select a station and add the **Signals Plus Station** extension
2. In the **Approach Signals** field, add signal IDs that should show caution for approaching trains
3. Trains scheduled to stop will now see caution on these signals

## Signal Aspects Quick Reference

| Aspect | When Displayed |
|--------|----------------|
| Idle | No train within braking distance |
| Pass | Train approaching, track ahead is clear |
| Caution | Signal ahead is Stop, or approaching scheduled station stop |
| Stop | Track block is occupied |


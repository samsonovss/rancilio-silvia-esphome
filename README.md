# Rancilio Silvia ESPHome

[Русская версия](README.ru.md)

An ESP32-S3, ESPHome, and Home Assistant modernization project for the Rancilio Silvia espresso machine.

The project controls machine power and boiler heating, measures boiler temperature with a PT100 and MAX31865, and provides separate PID modes for brewing and steaming.

> [!WARNING]
> The espresso machine contains hazardous mains voltage and a hot pressurized boiler. ESPHome is not a replacement for the original thermostat, thermal fuse, protective earth, or any other hardware safety device. Never work on the machine while it is connected to mains power.

## Features

- ESP32-S3 using ESP-IDF
- ESPHome and Home Assistant integration
- three-wire PT100 through MAX31865
- PID heater control through an SSR
- `Brew` and `Steam` modes
- adjustable target temperatures
- PID parameter controls and autotune from Home Assistant
- original power button support
- automatic shutdown timer
- status LED
- water-level sensor input
- 155 °C software overtemperature guard
- SSR lockout when the PT100 reading is invalid

## Repository layout

```text
.
├── README.md
├── README.ru.md
├── esphome/
│   ├── rancilio-silvia-power.yaml
│   └── secrets.example.yaml
├── docs/
│   ├── home-assistant.md
│   ├── safety.md
│   └── wiring.md
└── images/
```

## Quick start

1. Install ESPHome or the ESPHome Device Builder add-on in Home Assistant.
2. Copy `esphome/rancilio-silvia-power.yaml` into the ESPHome configuration directory.
3. Create `secrets.yaml` using `esphome/secrets.example.yaml`.
4. Verify every GPIO assignment and the electrical design for your exact board.
5. Validate the configuration before compiling the firmware.
6. Keep the machine under constant supervision during the first heater test.

## Default settings

| Setting | Value |
|---|---:|
| Brew temperature | 93 °C |
| Steam temperature | 140 °C |
| Software overtemperature limit | 155 °C |
| SSR control period | 10 seconds |
| Automatic shutdown | 60 minutes |

## Project status

This project is under active development. The water-level input is assigned to GPIO18, but the sensor output type is still being identified. Determine whether it uses an NPN/open-collector, PNP, or push-pull output and verify its voltage before connecting it to the ESP32-S3.

## Documentation

- [Wiring and GPIO](docs/wiring.md)
- [Home Assistant](docs/home-assistant.md)
- [Safety](docs/safety.md)

## License

No license has been granted yet. All rights are reserved by the author.

# Rancilio Silvia ESPHome

[Русская версия](README.ru.md)

[Telegram discussion group](https://t.me/Rancilio_Silvia) (Russian and English)

An ESP32-S3, ESPHome, and Home Assistant modernization project for the Rancilio Silvia espresso machine.

The project controls machine power and boiler heating, measures boiler temperature with a PT100 and MAX31865, and provides separate PID modes for brewing and steaming.

> [!WARNING]
> The espresso machine contains hazardous mains voltage and a hot pressurized boiler. ESPHome is not a replacement for the original thermostat, thermal fuse, protective earth, or any other hardware safety device. Never work on the machine while it is connected to mains power.


https://github.com/user-attachments/assets/92bf4580-1ab9-4535-a1f1-395bb5a3d315


## Features

- ESP32-S3 using ESP-IDF
- ESPHome and Home Assistant integration
- three-wire PT100 through MAX31865
- PID heater control through an SSR
- `Brew` and `Steam` modes
- adjustable target temperatures
- configurable brew-temperature offset applied to PID control
- PID parameter controls and autotune from Home Assistant
- automatic saving of successful autotune coefficients
- original power button support
- automatic shutdown timer
- status LED
- water-level sensor input
- configurable software overtemperature guard
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

## Configuration

The brew temperature, steam temperature, and automatic shutdown time are adjustable from Home Assistant. Values stored in the YAML file are initial defaults, not fixed machine specifications.

### Brew temperature model

`Silvia Brew Target` represents the desired estimated temperature at the coffee puck. In `Brew` mode, PID control uses:

```text
Estimated Brew Temperature = PT100 Boiler Temperature - Brew Temperature Offset
Brew Boiler Target = Brew Target + Brew Temperature Offset
```

For example, a brew target of `93 °C` with a `10 °C` offset produces a boiler target of approximately `103 °C`.

The PT100 entity always reports the unmodified temperature measured at the boiler. The offset is not applied in `Steam` mode, and the software overtemperature guard always uses the raw PT100 reading.

The estimated brew temperature is a model, not a direct water measurement. Keep the offset at `0 °C` until it has been calibrated at the group under realistic flow conditions.

The software overtemperature limit is a firmware setting. The heater SSR uses a one-second `slow_pwm` period; changing it may require PID retuning.

## Project status

This project is under active development. The XKC-Y25-NPN water-level sensor uses an active-low open-collector output on GPIO18 with the ESP32-S3 internal 3.3 V pull-up enabled.

## Documentation

- [Wiring and GPIO](docs/wiring.md)
- [Home Assistant](docs/home-assistant.md)
- [Safety](docs/safety.md)

## License

No license has been granted yet. All rights are reserved by the author.

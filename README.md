# Rancilio Silvia ESPHome Controller

[Русская версия](README.ru.md)

[Telegram discussion group](https://t.me/Rancilio_Silvia) (Russian and English)

Digital ESPHome controller for the Rancilio Silvia espresso machine, built on ESP32-S3 and integrated with Home Assistant.

The project is more than an external PID retrofit. The goal is to move the machine control logic to a digital controller: high-voltage loads are switched by relays/SSR, while the original front-panel controls become low-voltage GPIO inputs. This makes the machine controllable from Home Assistant and leaves room for a touchscreen or another digital interface later.

> [!WARNING]
> The espresso machine contains hazardous mains voltage and a hot pressurized boiler. ESPHome is not a replacement for the original thermostat, thermal fuse, protective earth, or any other hardware safety device. Never work on the machine while it is connected to mains power.

![dashboard](images/dashboard.png)

https://github.com/user-attachments/assets/92bf4580-1ab9-4535-a1f1-395bb5a3d315

## Current Status

The controller is already running on a real Rancilio Silvia. The hardware is still a prototype based on an ESP32-S3 development board and point-to-point wiring.

Current implementation:

- ESP32-S3 with ESP-IDF;
- ESPHome and Home Assistant integration;
- PT100 boiler temperature measurement through MAX31865;
- PID heater control through an SSR;
- `Brew` and `Steam` operating modes;
- automated brew shot with profiles and preinfusion;
- hot water and steam mode controls;
- water-level sensor input;
- automatic shutdown timer;
- shot counter, backflush reminder, and automated backflush program;
- configurable coffee dose and estimated dry coffee usage;
- PID autotune and automatic storage of successful coefficients.

The next hardware steps are pressure sensing and pump control for pressure profiling. A digital I2C pressure sensor is planned, and a one-channel RobotDyn AC dimmer has been ordered for pump control experiments.

## Hardware Overview

Main parts used by the current prototype:

- ESP32-S3 development board;
- three-wire PT100 sensor;
- MAX31865 RTD amplifier;
- SSR for the boiler heater;
- relay outputs for machine power, pump, and brew valve;
- low-voltage inputs for the original power, brew shot, hot water, and steam mode controls;
- XKC-Y25-NPN water-level sensor input.

Planned additions:

- digital I2C pressure sensor for brew pressure measurement and pressure profiling;
- one-channel RobotDyn AC dimmer for pump power control;
- dedicated PCB with pluggable connectors for sensors, relays, and peripheral devices.

## Features

### Temperature And Heating

- PID heater control through ESPHome `climate`;
- separate brew and steam targets;
- adjustable brew-temperature offset used by PID control;
- raw PT100 boiler temperature remains available separately;
- PID `KP`, `KI`, and `KD` controls from Home Assistant;
- PID autotune status and automatic saving of successful autotune coefficients;
- configurable software overtemperature guard;
- SSR lockout when the PT100 reading is invalid.

### Brewing

- automated timed brew shot;
- brew profiles: `Classic`, `Soft Preinfusion`, `Long Preinfusion`, and `Custom`;
- configurable preinfusion pump time, preinfusion pause, and main shot duration;
- live brew-shot phase status with countdown;
- physical low-voltage brew shot input;
- manual pump and brew-valve relay controls.

### Water, Steam, And Power

- machine power relay control;
- hot water mode;
- steam mode;
- physical low-voltage inputs for hot water and steam mode;
- inactivity-based automatic shutdown;
- auto-off timer reset on brew shot, hot water, steam mode, pump, and brew-valve activity;
- status LED;
- water-level monitoring and text status.

### Counters And Maintenance

- shot-count based backflush reminder;
- automated backflush program;
- manual backflush stop and counter reset;
- lifetime shot counter;
- configurable dry coffee dose per shot;
- estimated total dry coffee usage.

## Repository Layout

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

## Quick Start

1. Install ESPHome or the ESPHome Device Builder add-on in Home Assistant.
2. Copy `esphome/rancilio-silvia-power.yaml` into the ESPHome configuration directory.
3. Create `secrets.yaml` using `esphome/secrets.example.yaml`.
4. Verify every GPIO assignment and the electrical design for your exact board and relay modules.
5. Validate the configuration before compiling the firmware.
6. Keep the machine under constant supervision during the first heater, pump, and valve tests.

## Home Assistant Configuration

Most user-facing values are exposed as Home Assistant entities. Values stored in YAML are initial defaults, not fixed machine specifications.

Adjustable values include:

- brew target temperature;
- steam target temperature;
- brew-temperature offset;
- PID coefficients;
- brew profile;
- preinfusion pump time;
- preinfusion pause;
- main shot duration;
- backflush reminder threshold;
- dry coffee dose per shot;
- automatic shutdown time.

### Brew Shot And Profiles

`Silvia Brew Shot` starts an automated shot sequence:

1. open the brew valve;
2. optionally run the pump for preinfusion;
3. optionally pause after preinfusion;
4. run the pump for the configured shot duration;
5. stop the pump and close the brew valve.

`Silvia Brew Profile` provides presets and a custom mode:

- `Classic`: no preinfusion, 25 s shot;
- `Soft Preinfusion`: 2 s pump, 5 s pause, 25 s shot;
- `Long Preinfusion`: 3 s pump, 10 s pause, 28 s shot;
- `Custom`: selected automatically when timing values are edited manually.

`Silvia Brew Shot Status` reports the current automated shot phase and countdown. In the current configuration the published strings are localized:

- `Предсмачивание`: remaining preinfusion pump time;
- `Пауза`: remaining pause time after preinfusion;
- `Пролив`: remaining main brew time;
- `Ожидание`: no automated shot is running.

The dashboard can use this status as the primary live shot timer instead of inferring the phase from entity timestamps.

### Coffee Usage

`Silvia Coffee Dose Grams` stores the configured dry coffee dose per shot. The default is `14 g`, and the value can be changed from Home Assistant.

`Silvia Coffee Grounds Used` estimates total dry coffee usage:

```text
Coffee Grounds Used = Silvia Lifetime Shots × Silvia Coffee Dose Grams
```

This is an estimate of ground coffee consumption, not the beverage weight in the cup.

### Backflush Program

`Silvia Backflush Shots` counts completed automated shots since the last group backflush. `Silvia Lifetime Shots` keeps the total shot count and is not reset by cleaning.

`Silvia Backflush Reminder Shots` sets the reminder threshold. The default is `60` shots. Setting it to `0` disables the reminder. When the counter reaches the threshold, `Silvia Backflush Status` changes to a due state. The reminder is informational only and does not block brewing.

`Silvia Start Backflush` runs an automated pump and 3-way-valve cleaning sequence. `Silvia Stop Backflush` aborts the running sequence and turns off the pump and valve. `Silvia Reset Backflush Shots` manually clears the reminder counter.

### Brew Temperature Model

`Silvia Brew Target` represents the desired estimated temperature at the coffee puck. In `Brew` mode, PID control uses:

```text
Estimated Brew Temperature = PT100 Boiler Temperature - Brew Temperature Offset
Brew Boiler Target = Brew Target + Brew Temperature Offset
```

For example, a brew target of `93 °C` with a `10 °C` offset produces a boiler target of approximately `103 °C`.

The PT100 entity always reports the unmodified boiler temperature. The offset is not applied in `Steam` mode, and the software overtemperature guard always uses the raw PT100 reading.

The estimated brew temperature is a model, not a direct water measurement. Keep the offset at `0 °C` until it has been calibrated at the group under realistic flow conditions.

## Roadmap

- Add a digital I2C pressure sensor for brew pressure measurement.
- Use pressure feedback for pressure profiling.
- Experiment with the one-channel RobotDyn AC dimmer for pump power control.
- Update the Home Assistant dashboard for pressure graphs and profile controls.
- Design a dedicated PCB with proper connectors.
- Expand wiring and Home Assistant documentation.

## Documentation

- [Wiring and GPIO](docs/wiring.md)
- [Home Assistant](docs/home-assistant.md)
- [Safety](docs/safety.md)

## License

No license has been granted yet. All rights are reserved by the author.

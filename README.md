# Rancilio Silvia ESPHome Controller

[Русская версия](README.ru.md)

[Telegram discussion group](https://t.me/Rancilio_Silvia) (Russian and English)

An ESP32-S3, ESPHome, and Home Assistant digital controller project for the Rancilio Silvia espresso machine.

The project controls machine power, boiler heating, brew shot, hot water, and steam mode. It measures boiler temperature with a PT100 and MAX31865, provides separate PID modes for brewing and steaming, and exposes the machine to Home Assistant.

This is not just another external PID retrofit. The goal is a full digital replacement for the machine's control logic: the original mains-voltage control wiring is moved away from the front-panel switches, relays handle the high-voltage loads, and the original user controls become low-voltage GPIO inputs on the ESP32-S3. In principle, the physical switches can be removed entirely and replaced with a touchscreen or another digital user interface.

> [!WARNING]
> The espresso machine contains hazardous mains voltage and a hot pressurized boiler. ESPHome is not a replacement for the original thermostat, thermal fuse, protective earth, or any other hardware safety device. Never work on the machine while it is connected to mains power.

![dashboard](images/dashboard.png)

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
- full digital control of power, heater, pump, brew valve, hot water, and steam mode
- front-panel switches converted from mains control to low-voltage ESP32 GPIO inputs
- physical low-voltage inputs for brew shot, hot water, and steam mode
- Home Assistant controls for brew shot, hot water, steam mode, pump relay, and brew-valve relay
- timed brew shot control
- brew profiles: `Classic`, `Soft Preinfusion`, `Long Preinfusion`, and `Custom`
- configurable preinfusion pump time, preinfusion pause, and shot duration
- live brew-shot phase status with countdown for preinfusion, pause, and shot time
- shot-count based backflush reminder and automated backflush program
- configurable coffee dose and estimated coffee grounds usage based on completed shots
- manual pump and brew-valve relay control
- inactivity-based automatic shutdown timer
- auto-off timer reset on brew shot, hot water, steam mode, and manual pump/valve activity
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

The brew temperature, steam temperature, brew profile, preinfusion timings, shot duration, and automatic shutdown time are adjustable from Home Assistant. Values stored in the YAML file are initial defaults, not fixed machine specifications.

### Brew shot and profiles

`Silvia Brew Shot` starts an automated shot sequence:

1. open the brew valve;
2. optionally run the pump for preinfusion;
3. optionally pause after preinfusion;
4. run the pump for the configured shot duration;
5. stop the pump and close the brew valve.

The `Silvia Brew Profile` select provides three presets and a custom mode:

- `Classic`: no preinfusion, 25 s shot;
- `Soft Preinfusion`: 2 s pump, 5 s pause, 25 s shot;
- `Long Preinfusion`: 3 s pump, 10 s pause, 28 s shot;
- `Custom`: selected automatically when the timing numbers are edited manually.

`Silvia Hot Water` runs the pump without opening the brew valve. `Silvia Steam Mode` switches the PID target to the steam profile and returns to brew mode when turned off.

### Shot status and coffee usage

During an automated shot, `Silvia Brew Shot Status` reports the current phase and countdown. In the current configuration the published status strings are localized:

- `Предсмачивание`: remaining preinfusion pump time;
- `Пауза`: remaining pause time after preinfusion;
- `Пролив`: remaining main brew time;
- `Ожидание`: no automated shot is running.

The Home Assistant dashboard can use this status as the primary live shot timer instead of trying to infer the current phase from entity timestamps.

`Silvia Coffee Dose Grams` stores the configured dry coffee dose per shot. The default is `14 g`, and it can be adjusted from Home Assistant. `Silvia Coffee Grounds Used` estimates total dry coffee usage from the lifetime shot counter and the configured dose:

```text
Coffee Grounds Used = Silvia Lifetime Shots × Silvia Coffee Dose Grams
```

This is an estimate of ground coffee consumption, not the beverage weight in the cup.

### Backflush reminder and program

`Silvia Backflush Shots` counts completed automatic brew shots since the last group backflush. `Silvia Lifetime Shots` keeps the total shot count and is not reset by cleaning.

`Silvia Backflush Reminder Shots` sets the reminder threshold. The default is `60` shots and the value is adjustable from Home Assistant. Setting it to `0` disables the reminder. When the counter reaches the threshold, `Silvia Backflush Status` changes to a due state. The reminder is informational only; it does not block brewing.

`Silvia Start Backflush` runs an automated pump and 3-way-valve cleaning sequence. `Silvia Stop Backflush` aborts the running sequence and turns off the pump and valve. `Silvia Reset Backflush Shots` manually clears the reminder counter. The backflush counter is reset automatically only after the program finishes.

### Automatic shutdown

`Silvia Auto Off Minutes` is an inactivity timer. When the machine is powered on, the countdown starts. It restarts when brew shot, hot water, steam mode, pump relay, or brew-valve relay activity begins. Setting the value to `0` disables automatic shutdown.

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

## Project Status

The project is under active development but is already fully functional and running on a real Rancilio Silvia espresso machine.

The current implementation includes:

* PT100 temperature measurement through MAX31865;
* PID heater control via SSR;
* Brew and Steam operating modes;
* Home Assistant integration through ESPHome;
* automatic shutdown;
* water-level monitoring;
* PID autotune and parameter storage.

At the moment, the system is built as a prototype using an ESP32-S3 development board and point-to-point wiring. The original mains-voltage control wiring has been removed from the control switches during this conversion; high-voltage loads are now switched through the controller's relay outputs.

The next major milestone is the development of a dedicated PCB with pluggable connectors for sensors, relays, and peripheral devices. This will improve reliability, simplify assembly, and make the project easier to reproduce for other users.

Testing, hardware refinement, and documentation development are ongoing.

## Documentation

- [Wiring and GPIO](docs/wiring.md)
- [Home Assistant](docs/home-assistant.md)
- [Safety](docs/safety.md)

## License

No license has been granted yet. All rights are reserved by the author.

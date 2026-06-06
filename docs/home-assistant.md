# Home Assistant

После подключения ESPHome создаёт сущности для:

- питания кофемашины;
- температуры бойлера;
- мощности PID;
- режима `Brew` или `Steam`;
- целевых температур;
- коэффициентов `KP`, `KI` и `KD`;
- PID autotune;
- статуса PID autotune с временем, температурой и мощностью;
- таймера автоотключения;
- уровня воды и текстового статуса.

Названия сущностей формируются Home Assistant автоматически из имени устройства `rancilio-silvia-power`.

## Рекомендуемая панель

На отдельную карточку удобно вынести:

- `Silvia Power Relay`;
- `Silvia Brew PID`;
- `Silvia PID Mode`;
- `Silvia Boiler Temperature PT100`;
- `Silvia PID Heat Output`;
- `Silvia PID Autotune Status`;
- `Silvia Auto Off Remaining`;
- `Silvia Water Status`.

Не запускайте PID autotune без воды в бойлере и без постоянного контроля температуры.

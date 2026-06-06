# Home Assistant

После подключения ESPHome создаёт сущности для:

- питания кофемашины;
- температуры бойлера;
- расчётной температуры приготовления и её цели;
- настраиваемой температурной поправки;
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
- `Silvia Estimated Brew Temperature`;
- `Silvia Estimated Brew Target`;
- `Silvia Brew Temperature Offset`;
- `Silvia PID Heat Output`;
- `Silvia PID Autotune Status`;
- `Silvia Auto Off Remaining`;
- `Silvia Water Status`.

Не запускайте PID autotune без воды в бойлере и без постоянного контроля температуры.

## Расчётная температура приготовления

`Silvia Boiler Temperature PT100` показывает фактическую температуру в точке установки датчика. Расчётная температура определяется по формуле:

```text
Estimated Brew Temperature = Boiler Temperature - Brew Temperature Offset
```

Положительная поправка означает, что вода у кофейной таблетки холоднее показания PT100. Начальное значение поправки равно `0 °C`; изменяйте его только после измерения температуры у группы на полностью прогретой машине при обычном расходе воды.

Расчётный сенсор предназначен для отображения и не участвует в PID-регулировании или защите от перегрева.

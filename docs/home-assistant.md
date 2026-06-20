# Home Assistant

После подключения ESPHome создаёт сущности для:

- питания кофемашины;
- температуры бойлера;
- расчётной температуры приготовления и её цели;
- настраиваемой температурной поправки;
- мощности PID;
- режима `Brew` или `Steam`;
- профилей пролива;
- длительности пролива и параметров предсмачивания;
- целевых температур;
- коэффициентов `KP`, `KI` и `KD`;
- PID autotune;
- статуса PID autotune с временем, температурой и мощностью;
- таймера автоотключения по неактивности;
- физических кнопок `Brew Shot`, `Hot Water` и `Steam Mode`;
- управления проливом, горячей водой, режимом пара, помпой и brew-клапаном;
- уровня воды и текстового статуса.

Названия сущностей формируются Home Assistant автоматически из имени устройства `rancilio-silvia-power`.

## Рекомендуемая панель

На отдельную карточку удобно вынести:

- `Silvia Power Relay`;
- `Silvia Brew PID`;
- `Silvia PID Mode`;
- `Silvia Brew Profile`;
- `Silvia Boiler Temperature PT100`;
- `Silvia Estimated Brew Temperature`;
- `Silvia Estimated Brew Target`;
- `Silvia Brew Boiler Target`;
- `Silvia Brew Temperature Offset`;
- `Silvia Brew Shot Seconds`;
- `Silvia Preinfusion Pump Seconds`;
- `Silvia Preinfusion Pause Seconds`;
- `Silvia PID Heat Output`;
- `Silvia PID Autotune Status`;
- `Silvia Auto Off Remaining`;
- `Silvia Brew Shot`;
- `Silvia Hot Water`;
- `Silvia Steam Mode`;
- `Silvia Brew Shot Button`;
- `Silvia Hot Water Button`;
- `Silvia Brew Steam Mode Button`;
- `Silvia Pump Relay`;
- `Silvia Brew Valve Relay`;
- `Silvia Water Status`.

Не запускайте PID autotune без воды в бойлере и без постоянного контроля температуры.

## Расчётная температура приготовления

`Silvia Boiler Temperature PT100` показывает фактическую температуру в точке установки датчика. Расчётная температура определяется по формуле:

```text
Estimated Brew Temperature = Boiler Temperature - Brew Temperature Offset
```

В режиме `Brew` PID регулирует расчётную температуру воды. Поэтому физическая цель бойлера определяется так:

```text
Brew Boiler Target = Brew Target + Brew Temperature Offset
```

Например, при цели приготовления `93 °C` и поправке `10 °C` PID нагревает бойлер примерно до `103 °C`. В режиме `Steam` поправка не применяется. Фактическая температура PT100 всегда остаётся доступна отдельным сенсором, а защита от перегрева работает только по ней.

Положительная поправка означает, что вода у кофейной таблетки холоднее показания PT100. Начальное значение поправки равно `0 °C`; изменяйте его только после измерения температуры у группы на полностью прогретой машине при обычном расходе воды. Поправка участвует в PID-регулировании только в режиме `Brew`; защита от перегрева всегда использует исходную температуру PT100.

## Пролив, горячая вода и пар

`Silvia Brew Shot` запускает автоматический пролив по текущим настройкам профиля. В начале открывается brew-клапан, затем выполняется предсмачивание, пауза и основной пролив. После завершения помпа и клапан выключаются.

`Silvia Hot Water` включает помпу без открытия brew-клапана. Это рассчитано на использование штатного парового крана/трубки для выдачи воды.

`Silvia Steam Mode` переключает PID в режим `Steam`; при выключении возвращает `Brew`.

`Silvia Brew Profile` управляет значениями `Silvia Preinfusion Pump Seconds`, `Silvia Preinfusion Pause Seconds` и `Silvia Brew Shot Seconds`. Если эти числа изменить вручную, профиль автоматически становится `Custom`.

## Автоотключение

`Silvia Auto Off Minutes` — таймер неактивности, а не таймер "от момента включения любой ценой". Отсчёт перезапускается при включении пролива, горячей воды, режима пара, помпы или brew-клапана. Значение `0` отключает автоотключение.

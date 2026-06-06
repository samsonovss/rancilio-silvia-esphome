# Подключение / Wiring

## Назначение GPIO

| Функция | GPIO | Примечание |
|---|---:|---|
| Силовое реле машины | GPIO4 | Активный низкий уровень |
| Штатная кнопка Power | GPIO5 | Вход с внутренней подтяжкой |
| Статусный LED | GPIO6 | Цифровой выход |
| SSR нагревателя | GPIO7 | `slow_pwm`, период 1 с |
| SPI CLK | GPIO11 | MAX31865 |
| SPI MISO | GPIO12 | MAX31865 |
| SPI MOSI | GPIO13 | MAX31865 |
| MAX31865 CS | GPIO14 | Выбор устройства SPI |
| Датчик уровня воды | GPIO18 | Тип выхода пока уточняется |

## PT100 и MAX31865

Конфигурация рассчитана на трёхпроводной PT100, опорный резистор 430 Ом и сетевой фильтр 50 Гц. Проверьте перемычки и фактический опорный резистор своей платы MAX31865.

## Датчик воды

GPIO ESP32-S3 рассчитаны на 3,3 В и не являются устойчивыми к 5 В. Нельзя подключать к GPIO сигнальный выход неизвестного напряжения.

Перед окончательной настройкой выясните:

- напряжение питания датчика;
- максимальное напряжение на сигнальном выходе;
- тип выхода: NPN/open-collector, PNP или push-pull;
- активный уровень при наличии воды.

Для NPN/open-collector обычно нужна подтяжка сигнала к 3,3 В, после чего может потребоваться `inverted: true`. Для PNP-выхода потребуется согласование уровней.

## Mains wiring

The mains and low-voltage sections must be physically separated. Use correctly rated relays, SSRs, terminals, wire, insulation, fusing, and protective earth. The exact mains wiring is intentionally not documented until the assembled machine has been verified.

markdown
# ESPHome конфигурация для дисплея VIEWESMART UEDX48480021-MD80ESP32-2.1

[![ESPHome](https://img.shields.io/badge/ESPHome-2024.x-000000?logo=esphome&logoColor=white)](https://esphome.io/)
[![ESP32-S3](https://img.shields.io/badge/MCU-ESP32--S3-E7352C?logo=espressif&logoColor=white)](https://www.espressif.com/en/products/socs/esp32-s3)
[![Framework](https://img.shields.io/badge/Framework-esp--idf-4B5563?logo=espressif&logoColor=white)](https://docs.espressif.com/projects/esp-idf/)
[![Display](https://img.shields.io/badge/Display-480%C3%97480%20RGB-1E90FF)](https://www.viewesmart.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Language](https://img.shields.io/badge/lang-RU%20%7C%20EN-blue)](#)
[![README EN](https://img.shields.io/badge/README-English-blue)](./README.en.md)

Конфигурация дисплея **VIEWESMART UEDX48480021-MD80ESP32-2.1** (480×480, 2.1") с энкодером и кнопкой в **ESPHome**.

Проект собран и протестирован на плате **Espressif ESP32-S3-DevKitC-1-N16R8V** (16 MB Flash Quad, 8 MB PSRAM Octal).

<p align="center">
  <img src="./docs/images/display.jpg" alt="VIEWESMART UEDX48480021-MD80ESP32-2.1" width="480"/>
  <br/>
  <em>Дисплей VIEWESMART UEDX48480021-MD80ESP32-2.1 (480×480, MIPI RGB)</em>
</p>

---

## 📸 Скриншоты интерфейса LVGL

<table>
  <tr>
    <td align="center">
      <img src="./docs/images/page_spinners.png" width="240" alt="Spinners"/><br/>
      <b>1. Spinners</b><br/><sub>Два вращающихся индикатора</sub>
    </td>
    <td align="center">
      <img src="./docs/images/page_arc.png" width="240" alt="Arc"/><br/>
      <b>2. Arc</b><br/><sub>Редактируемое значение 0–100</sub>
    </td>
    <td align="center">
      <img src="./docs/images/page_text.png" width="240" alt="Text"/><br/>
      <b>3. Text</b><br/><sub>Статичный текст «ПРИВЕТ!!!»</sub>
    </td>
  </tr>
</table>

> 💡 Поместите скриншоты в `docs/images/`. Достаточно одной фотографии дисплея на каждой странице.

---

## 📋 Аппаратное обеспечение

| Компонент | Описание |
|---|---|
| Микроконтроллер | ESP32-S3 (N16R8V — 16 MB Flash, 8 MB PSRAM Octal) |
| Дисплей | VIEWESMART UEDX48480021-MD80ESP32-2.1 (MIPI RGB 480×480, 18-bit) |
| Энкодер | Механический инкрементальный (GPIO5 / GPIO6) |
| Кнопка | Тактовая кнопка энкодера или отдельная (GPIO0) |

**Подключение GPIO ESP32-S3 выполнено согласно даташиту производителя:**
📄 [`UEDX48480021-MD80E-V3.2-SPEC.pdf`](./UEDX48480021-MD80E-V3.2-SPEC.pdf)

---

## 🔌 Распиновка (Pinout)

### Дисплей (MIPI RGB)

| Сигнал | GPIO | Примечание |
|---|---|---|
| CS | GPIO18 | Chip Select |
| RESET | GPIO8 | Сброс дисплея |
| DE | GPIO17 | Data Enable |
| HSYNC | GPIO46 | Горизонтальная синхронизация |
| VSYNC | GPIO3 | Вертикальная синхронизация |
| PCLK | GPIO9 | Pixel Clock (16 MHz, inverted) |
| **RED** | GPIO40, 41, 42, 2, 1 | R0…R4 (5 бит) |
| **GREEN** | GPIO21, 47, 48, 45, 38, 39 | G0…G5 (6 бит) |
| **BLUE** | GPIO10, 11, 12, 13, 14 | B0…B4 (5 бит) |

### Подсветка и периферия

| Сигнал | GPIO | Примечание |
|---|---|---|
| Backlight (LEDC) | GPIO7 | PWM 150 Hz, инвертирован |
| Encoder A | GPIO6 | Input Pull-up, debounce 0.15 s |
| Encoder B | GPIO5 | Input Pull-up, debounce 0.15 s |
| Кнопка (SW_KEY) | GPIO0 | Input Pull-up, `delayed_on/off: 30 ms` |

> ⚠️ Пины **GPIO12** и **GPIO13** используются одновременно программной SPI-шиной и как линии данных Blue (B2, B3). В конфиге это разрешено через `allow_other_uses: true`.

---

## 🧩 Блок инициализации дисплея и настройка SPI

### Программная SPI-шина

Так как линии данных RGB-дисплея пересекаются с пинами SPI, используется **software SPI**:

```yaml
spi:
  clk_pin:
    number: GPIO13
    allow_other_uses: true
  mosi_pin:
    number: GPIO12
    allow_other_uses: true
  interface: software
  id: spi_software
Параметры RGB-панели
yaml
display:
  - platform: mipi_rgb
    model: custom
    id: my_display
    dimensions:
      width: 480
      height: 480
    hsync_pulse_width: 10
    hsync_back_porch: 40
    hsync_front_porch: 50
    vsync_pulse_width: 10
    vsync_back_porch: 40
    vsync_front_porch: 20
    pclk_frequency: 16MHz
    pclk_inverted: true
    pixel_mode: 18bit
    color_order: BGR
    invert_colors: true
Последовательность инициализации (init_sequence)
Дисплей инициализируется через набор команд (ST7701-подобный контроллер). Полный init_sequence включает:

Команды разблокировки 0xF0 0x55 0xAA 0x52 0x08 0x00

Настройки питания (0xC1, 0xC9, 0xAC, 0xA7, 0xA0)

GIP / timing (0x87, 0x86, 0xFA)

Гамма-кривые (0x60–0x67, 0xD1–0xD6)

Выход из sleep 0x11 → задержка 120 ms

Включение дисплея 0x29 → задержка 20 ms

Полная таблица команд приведена в esp32-s3-display.yaml.

🎛️ Управление
Энкодер — две логики работы
Логика переключается глобальной переменной edit_mode (изменяется кнопкой на GPIO0, только на странице Arc):

Режим	Вращение энкодера
Навигация (edit_mode = false)	Переключение между 3 страницами LVGL
Редактирование (edit_mode = true, только стр. Arc)	Изменение значения дуги (шаг ±2, диапазон 0…100)
Страницы LVGL
page_spinners — два вращающихся индикатора (демонстрация LVGL)

page_arc — дуговая шкала с редактируемым значением (кнопка переключает режим редактирования)

page_text — статичный текст «ПРИВЕТ!!!»

🚀 Быстрый старт
Склонируйте репозиторий и откройте esp32-s3-display.yaml в ESPHome.

Создайте файл secrets.yaml со своими данными:

yaml
wifi_ssid: "YourWiFi"
wifi_password: "YourPassword"
Укажите свой api.encryption.key (сгенерировать можно в ESPHome Dashboard).

Прошейте плату через USB (первый раз) или OTA (последующие).

bash
esphome run esp32-s3-display.yaml
🛠️ Известные проблемы / Troubleshooting
<details> <summary><b>❌ Дисплей чёрный / нет изображения</b></summary>
Убедитесь, что в esp32: выбран framework: esp-idf — Arduino не поддерживает MIPI RGB на ESP32-S3.

Проверьте, что psram: mode: octal, speed: 80MHz включена. Без PSRAM буфер 480×480 не поместится.

Убедитесь, что подсветка включена: light: display_backlight в состоянии ON (restore_mode: ALWAYS_ON).

Проверьте, что сигнал DE (GPIO17) и PCLK (GPIO9) действительно доходят до дисплея.

Попробуйте добавить - delay 150ms после [0x11] в init_sequence.

</details><details> <summary><b>🎨 Неправильные цвета (инверсия, красное ↔ синее)</b></summary>
Поиграйте с color_order: BGR ⇄ RGB.

Проверьте invert_colors: true ⇄ false.

Убедитесь в правильном порядке битов в группах red/green/blue — от старшего к младшему (или наоборот, зависит от модуля).

</details><details> <summary><b>📺 Артефакты, мерцание, «снег» на изображении</b></summary>
Снизьте pclk_frequency с 16MHz до 12MHz или 9MHz — самый частый источник проблем.

Проверьте длины проводов — PCLK и DATA должны быть как можно короче.

Убедитесь, что buffer_size: 20% — при нехватке памяти попробуйте увеличить до 25% (при наличии PSRAM).

Отключите logger: level: VERBOSE — вывод в UART отбирает такты у DMA.

</details><details> <summary><b>🔄 Энкодер «дёргается» или пропускает шаги</b></summary>
Увеличьте debounce в фильтре до 0.2s.

Убедитесь в наличии подтягивающих резисторов (pullup: true) или добавьте внешние 10 кОм к 3.3 В.

Проверьте, что GPIO5/GPIO6 не заняты другими компонентами.

</details><details> <summary><b>⚠️ Ошибка компиляции «pin is already used»</b></summary>
GPIO12 и GPIO13 используются дважды (SPI + Blue data). У обоих должен быть флаг allow_other_uses: true:

yaml
- number: GPIO12
  allow_other_uses: true
Это же относится к spi.mosi_pin и spi.clk_pin.

</details><details> <summary><b>💾 Не хватает памяти / OOM при старте</b></summary>
Уменьшите buffer_size в lvgl: до 15% или 10%.

Уберите неиспользуемые шрифты (font_20 не задействован в виджетах).

Включите esp32.framework.sdkconfig с CONFIG_SPIRAM_USE_MALLOC=y.

</details><details> <summary><b>🔌 OTA не работает после первой прошивки</b></summary>
Первая прошивка обязательно по USB (кабелем данных, не «charge-only»).

Для OTA убедитесь, что ota: platform: esphome присутствует и устройство в той же сети.

Проверьте, что api.encryption.key совпадает в конфиге и в HA/ESPHome Dashboard.

</details>
📁 Структура репозитория
text
.
├── esp32-s3-display.yaml                # Основная конфигурация
├── secrets.yaml                         # (не коммитить!) Секреты Wi-Fi и API
├── display/
│   └── fonts/
│       └── Roboto-Regular.ttf           # Шрифт с кириллицей
├── UEDX48480021-MD80E-V3.2-SPEC.pdf     # Даташит производителя
├── README.md                            # Этот файл (RU)
├── README.en.md                         # English version
├── .gitignore
└── LICENSE                              # MIT
📝 Примечания
Используется esp-idf framework (не Arduino) — обязательно для стабильной работы RGB-дисплея и PSRAM Octal.

PSRAM настроена как octal @ 80 MHz — это критично для буферизации 480×480 RGB.

Буфер LVGL: buffer_size: 20% — компромисс между FPS и потреблением RAM.

Частота pclk_frequency: 16 MHz подобрана под данный модуль; при артефактах изображения — уменьшайте.

📜 Лицензия
Проект распространяется под лицензией MIT. См. LICENSE.

🙏 Благодарности
ESPHome — за превосходный фреймворк

VIEWESMART — за дисплейные модули

LVGL — за графическую библиотеку


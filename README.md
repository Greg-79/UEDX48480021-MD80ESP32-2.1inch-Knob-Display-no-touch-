# ESPHome конфигурация для дисплея VIEWESMART UEDX48480021-MD80ESP32-2.1

Конфигурация дисплея **VIEWESMART UEDX48480021-MD80ESP32-2.1** (480×480, 2.1") с энкодером и кнопкой в **ESPHome**.

Проект собран и протестирован на плате **Espressif ESP32-S3-DevKitC-1-N16R8V** (16 MB Flash Quad, 8 MB PSRAM Octal).

---

## 📋 Аппаратное обеспечение

| Компонент | Описание |
|---|---|
| Микроконтроллер | ESP32-S3 (N16R8V — 16 MB Flash, 8 MB PSRAM Octal) |
| Дисплей | VIEWESMART UEDX48480021-MD80ESP32-2.1 (MIPI RGB 480×480, 18-bit) |
| Энкодер | Механический инкрементальный (GPIO5 / GPIO6) |
| Кнопка | Тактовая кнопка энкодера или отдельная (GPIO0) |

**Подключение GPIO ESP32-S3 выполнено согласно даташиту производителя:**
📄 `UEDX48480021-MD80E-V3.2-SPEC.pdf` — *(положите файл в корень репозитория или добавьте ссылку)*

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

# Heltec V4 868: Heltec WiFi LoRa 32 V4.3 на 868 МГц

Плата **Heltec WiFi LoRa 32 V4.3**: ESP32-S3, SX1262, радіомодуль KCT8103L (підсилювач на передачу і LNA на прийом). Прошивки є для двох версій: з OLED і з TFT-екраном. Усередині наші патчі (як у FakeSolar і Faketec RA62):
- **скидання AGC:**
  - очікування калібрування перед записом регістрів;
  - пропуск скидання, поки йде прийом;
- **регістр 0x8B5**, TCXO увімкнений у standby;
- **фільтр рівня шуму:** без вибірок одразу після старту прийому;
- **GPS:**
  - автовизначення швидкості порту;
  - захист від спуфінгу позиції й часу (`GPS_GUARD`);
  - цикл живлення: раз на 30 хв, до 5 хв увімкнено;
- **сенсори** визначаються автоматично;
- **дільник батареї** задається з CLI на всіх ролях, включно з компаньйоном: `set adc.multiplier <x>` / `get adc.multiplier`, `0` = значення плати (5.42).

**Радіо:** 869.618 МГц / BW 62.5 кГц / SF8 / **CR8**, це пресет «EU/UK (Narrow)» у застосунку. Апстрімні збірки без `LORA_CR` мають CR5, а тут CR8 задано явно.

**Потужність:** як в апстрімі. За замовчуванням `LORA_TX_POWER=10` на SX1262, що за коментарем Heltec дає близько 22 dBm на виході KCT8103L (заміри в MeshCore #1708: вихід насичується на рівні близько 27–28 dBm при `set tx` 18–20). `set tx` понад 10 піднімає вихід ще вище, тож враховуйте ліміт вашої країни: у смузі 869.4–869.65 МГц у ЄС це 500 мВт ERP (27 dBm) з duty cycle 10%.

**LNA на прийом:** за замовчуванням вимкнений (bypass). Вмикається командою `set radio.fem.rxgain on`, стан показує `get radio.fem.rxgain`. Дає до ~17 дБ підсилення на прийомі, але в шумному ефірі може погіршити прийом, тому перед вмиканням подивіться на рівень шуму.

**Межі `GPS_GUARD`:** широта 44.0–52.5, довгота 22.0–40.3 (як у FakeSolar). Фікс поза цими межами відкидається.

> На залізі ще не перевірено.

## Файли

| Файл | Роль |
|---|---|
| `Heltec_V4_868_repeater*` | repeater, ім'я `Heltec Repeater`, пароль адміністратора `password` |
| `Heltec_V4_868_room_server*` | room server, ім'я `Heltec Room`, адмін `password`, гостьовий `hello` |
| `Heltec_V4_868_companion_radio_ble*` | companion BLE, PIN показується на екрані |
| `Heltec_V4_868_sensor*` | sensor, ім'я `Heltec v4 Sensor`, пароль `password` |
| `Heltec_V4_868_tft_*` | те саме для V4 з TFT-екраном |

- `*-merged.bin` — повна прошивка з нуля: веб-флешер MeshCore (Custom firmware) або `esptool.py --chip esp32s3 write_flash 0x0 <файл>-merged.bin`.
- `*.bin` — оновлення поверх уже встановленої прошивки MeshCore (OTA: `start ota`, або веб-флешер без стирання).

## Збірка

Конфігурація: `variants/heltec_v4_868/platformio.ini`. Варіант розширює апстрімні env `heltec_v4_*` нашими прапорцями.

```
pio run -e Heltec_V4_868_repeater -e Heltec_V4_868_room_server -e Heltec_V4_868_companion_radio_ble -e Heltec_V4_868_sensor
pio run -e Heltec_V4_868_tft_repeater -e Heltec_V4_868_tft_room_server -e Heltec_V4_868_tft_companion_radio_ble -e Heltec_V4_868_tft_sensor
```

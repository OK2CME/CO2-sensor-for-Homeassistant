# CO2-sensor-for-Homeassistant
CO2 sensor based on SCD41 with eInk on ESP32 for Homeassistant

Battery-Powered CO₂ Sensor with ESPHome

This project is a low-power battery-operated CO₂ sensor based on ESP32, Sensirion SCD41, and an ePaper display, fully integrated with ESPHome and Home Assistant.

It is designed for long battery life, reliable measurements, and remote control without unnecessary wake-ups.

I have tested with 2x 18650 in paralell (2 x 3200 mAh) with battery life 3-4 months. 

*** Components: ***
 - SCD41 CO2, T, RH sensor:
   https://www.aliexpress.com/item/1005009740863220.html?spm=a2g0o.productlist.main.1.7a321b7cy6NkQS&algo_pvid=8f1e427e-dc0b-40c6-9d02-d9a72af58082&algo_exp_id=8f1e427e-dc0b-40c6-9d02-d9a72af58082-0&pdp_ext_f=%7B%22order%22%3A%2219%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21USD%2132.46%2122.72%21%21%21224.33%21157.03%21%40211b6c1917700678619894225ea011%2112000050008202905%21sea%21CZ%21895548865%21X%211%210%21n_tag%3A-29919%3Bd%3A2d5fa0bd%3Bm03_new_user%3A-29895&curPageLogUid=Td3h7iEjNyMO&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005009740863220%7C_p_origin_prod%3A

 - Lolin D32 board:
 https://www.aliexpress.com/item/1005005476182210.html?spm=a2g0o.productlist.main.1.24e0A9T1A9T1kg&algo_pvid=030559df-eefc-40dd-9f21-9d04451b55d2&algo_exp_id=030559df-eefc-40dd-9f21-9d04451b55d2-0&pdp_ext_f=%7B%22order%22%3A%22126%22%2C%22spu_best_type%22%3A%22order%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21USD%214.34%214.34%21%21%214.34%214.34%21%402103856417700678999443360eaaaa%2112000041163128652%21sea%21CZ%21895548865%21X%211%210%21n_tag%3A-29919%3Bd%3A2d5fa0bd%3Bm03_new_user%3A-29895&curPageLogUid=Nk3MqKlkM9HT&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005005476182210%7C_p_origin_prod%3A


  - 2,13in WeActStudio epaper display: https://www.aliexpress.com/item/1005004644515880.html?spm=a2g0o.productlist.main.4.4864sYBbsYBb3C&algo_pvid=edba6ddd-6b41-4b92-844d-86b67024a802&algo_exp_id=edba6ddd-6b41-4b92-844d-86b67024a802-3&pdp_ext_f=%7B%22order%22%3A%222004%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21USD%219.87%219.15%21%21%2168.18%2163.19%21%4021038e6617700718789514887e86dc%2112000034319637606%21sea%21CZ%21895548865%21X%211%210%21n_tag%3A-29919%3Bd%3A2d5fa0bd%3Bm03_new_user%3A-29895&curPageLogUid=tdM6oFL3mkeG&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005004644515880%7C_p_origin_prod%3A
  - 2x 1M resistor for battery voltage measurement (voltage divider between BATT - D35 - GND)
  - LiIon battery holder - I used 2x 18650 liion 

Features

* SCD41 sensor
    - CO₂, temperature, and humidity measurement
    - Manual and calibration activation from HomeAssistant

* ePaper display
    - Displays measured values with zero power consumption when idle
    - Refreshes only when needed
    - Dedicated Low Battery screen
    - Ultra-low power operation

* Deep sleep between measurements
    - default 5 minutes deep sleep beteween measurements and display updates
    - Home Assistant connection is done every 15 minutes fot battery saving
* OTA update mode:
    - activated via Home Assistant input_boolean (Deep sleep is temporarily blocked)
    - Automatic timeout (e.g. 15 minutes)

Wi-Fi and CO₂ measurement disabled when battery voltage is low.
Only battery voltage monitoring remains active in low-battery mode.
Low-battery detection based on voltage.
Automatic recovery when charging is detected (voltage rising).
Asynchronous control from Home Assistant.


*** Device behavior ***

* Normal mode
deep sleep 5 min
Wake up → measure → update display → send data (every 3rd wake up) → deep sleep
  - When CO2 is over the 1000 ppm you will see the "!" warning sign.
  - When CO2 is over 1500 ppm you will see the inverted "!" warning sign. 

* Low battery mode
deep sleep 15 min
only voltage measurement
CO₂ measurement and Wi-Fi disabled
Display shows only low-battery indicator

* OTA mode
Deep sleep disabled
Allows safe firmware updates
Automatically returns to normal operation after timeout (15 mins)

### Home Assistant integration ###
Uses input_boolean helpers for:
- OTA mode: input_boolean.co2_sensor1_ota_mode
- Calibration request: input_boolean.co2_sensor1_manual_cal
  (for calibration is recomended at least 15 mins at fresh air)
Device connects to HA only once a 15 mins. It means if you change helper state, the sensor will update its state at next period. It may took up to 15 minutes. 
So if you want to do the OTA FW upgrade, you need to activate the helper and wait for the next period. You will see the values of CO2 and the others are updating in HA. Now you have 15 mins to update FW from ESPHome. After the timeout the sensor will reset the helper and return to the normal mode and go to sleep.

If you activate the Manual Calibration helper the situation is like in the OTA mode. Sensor will read the helper state at the next period. Caliibration will be done immediately after the connection to HA. So it needs to be at the fresh air at this time. The manual calibration helper should reset after the calibration. 

ESPHome device must be trusted to call HA services. If it is not, it will not be able to reset the helpers after actions. 

*** Use case ***
  - Indoor air quality monitoring
  - Bedrooms, living rooms, offices

# ESPHome smart doorbell

This is a ESP32Cam based project to create a smart doorbell. Since my last doorbell broke after two years in use, I wanted to create something smart, relatively cheap and cloud/subscription free. Since I love DIY projects started to search and mixed many projects to attain my needs.

The ideia was to have:
  - Camera to watch video stream and send a pic when triggered
  - Speaker to send some feedback and the ringing itself
  - Fingerprint scanner so I could use to unlock the garden gate without keys.

The project had some modifications along the way due to difficulties with the hardware and/or functionality.
  - Needed to add a light so I could see when someone ring the bell and its dark
  - Initially the idea was to use the fingerprint sensor as bell button. When the fingerprint were recognized it would unlock the gate, when there was not the bell would ring, but the time needed with the finger on the sensor would make people not to trigger the bell. so I went to add a fisical momentary button to trigger the bell.


# DESCRIPTION

The project was based on a mix of the ideas on the projects below, thank  you all for sharing:

For the basic idea, triggers and automations:
- https://tristam.ie/2023/758/

For the fingerprint sensor configuration and wiring:
- https://community.home-assistant.io/t/garage-fingerprint-sensor/312977
- https://www.youtube.com/watch?v=PuouLWa_vWo

For the audio response and ringing:
- https://github.com/ronschaeffer/video_doorbell_voice_response
  
For the flash light:
- https://www.makerguides.com/drive-power-leds-with-ld1500sb-esp32/ 

# Parts Used:
(I will put links to the parts but im not associated with any of these brands or shops, its just to show the specific parts used)

- Esp32cam with external antenna [Like this one](https://www.aliexpress.com/item/1005005958341078.html?spm=a2g0o.order_list.order_list_main.56.25eccaa4atuoRo)
- OV2640 Camera Module Fisheye Wide-angle Lens 120º [like this one](https://www.aliexpress.com/item/1005004779499695.html?spm=a2g0o.order_list.order_list_main.46.25eccaa4atuoRo)
- R503 fingerprint sensor [This one](https://www.aliexpress.com/item/33053655412.html?spm=a2g0o.order_list.order_list_main.41.25eccaa4atuoRo)
- DF Player Mini [Like this one](https://www.aliexpress.com/item/1005005656568976.html?spm=a2g0o.order_list.order_list_main.51.25eccaa4atuoRo)
- Small Speaker (reused from the previous doorbell)
- LD1500SB Led Driver [Like this one](https://www.aliexpress.com/item/1005005884585581.html?spm=a2g0o.productlist.main.1.ae3dNCBMNCBM2d&algo_pvid=90c72317-c73d-4103-8c3e-eaeac9e074a8&algo_exp_id=90c72317-c73d-4103-8c3e-eaeac9e074a8-0&pdp_npi=4%40dis%21EUR%217.15%217.15%21%21%2154.81%2154.81%21%402103868a17235603818416488e0e7c%2112000034698374434%21sea%21PT%21855154709%21X&curPageLogUid=3P6EwUmtQ5VH&utparam-url=scene%3Asearch%7Cquery_from%3A)
- High Power led bead [This one](https://www.aliexpress.com/item/1005006758358960.html?spm=a2g0o.order_list.order_list_main.15.25eccaa4atuoRo&gatewayAdapt=glo2bra)
- 3D Printed case mount all the parts. [Check it here](https://github.com/tmsaavedra/ESPHOME_Doorbell/blob/main/Doorbell_new_ver%20v20.stl)


# Wiring:
![alt text](https://github.com/tmsaavedra/ESPHOME_Doorbell/blob/main/wiring.png)

# Code:

There are some details you should modify like:
 - Wifi names and passwords
 - encriptions keys
 -   
Esp32cam yaml:

```yaml
esphome:
  name: espcam
  friendly_name: ESPCAM

esp32:
  board: esp32dev
  framework:
    type: arduino

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "YOUR_ENCRIPTION_KEY"
  services:
    - service: enroll
      variables:
        finger_id: int
        num_scans: int
      then:
        - fingerprint_grow.enroll:
            finger_id: !lambda 'return finger_id;'
            num_scans: !lambda 'return num_scans;'
        - fingerprint_grow.aura_led_control:
            state: ALWAYS_ON
            speed: 0
            color: PURPLE
            count: 0
    - service: cancel_enroll
      then:
        - fingerprint_grow.cancel_enroll:
    - service: delete
      variables:
        finger_id: int
      then:
        - fingerprint_grow.delete:
            finger_id: !lambda 'return finger_id;'
    - service: delete_all
      then:
        - fingerprint_grow.delete_all:
    - service: dfplayer_play
      variables:
        file: int
      then:
        - dfplayer.play: !lambda 'return file;'

    - service: dfplayer_play_loop
      variables:
        file: int
        loop_: bool
      then:
        - dfplayer.play:
            file: !lambda 'return file;'
            loop: !lambda 'return loop_;'
    - service: dfplayer_set_volume
      variables:
        volume: int
      then:
        - dfplayer.set_volume: !lambda 'return volume;'
    - service: dfplayer_stop
      then:
        - dfplayer.stop

ota:
  platform: esphome
  password: "YOUR_ENCRIPTION_KEY"

wifi:
  ssid: "YOUR_SSID"
  password: "YOUR_WIFI_PASSWORD"
  
  manual_ip:
    static_ip: 192.168.1.45
    gateway: 192.168.1.1
    subnet: 255.255.255.0

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Espcam Fallback Hotspot"
    password: "YOUR_ENCRIPTION_KEY"

captive_portal:

esp32_camera:
  id: espcam
  name: esp-cam
  external_clock:
    pin: GPIO0
    frequency: 10MHz
  i2c_pins:
    sda: GPIO26
    scl: GPIO27
  data_pins: [GPIO5, GPIO18, GPIO19, GPIO21, GPIO36, GPIO39, GPIO34, GPIO35]
  vsync_pin: GPIO25
  href_pin: GPIO23
  pixel_clock_pin: GPIO22
  power_down_pin: GPIO32
  resolution: 640x480
  jpeg_quality: 30  # max. 63
  max_framerate: 25.0fps
  idle_framerate: 0.5fps
  vertical_flip: true
  horizontal_mirror: false
  brightness: 2 # -2 to 2
  contrast: 1 # -2 to 2
  special_effect: none
  # exposure settings
  aec_mode: auto
  aec2: false
  ae_level: 0
  aec_value: 300
  # gain settings
  agc_mode: auto
  agc_gain_ceiling: 2x
  agc_value: 0
  # white balance setting
  wb_mode: auto
output:
# white LED
  - platform: ledc
    channel: 2
    pin: GPIO4
    id: espCamLED

  - platform: ledc
    pin: GPIO13
    id: gpio_13
    frequency: 1000Hz

# red status light
  - platform: gpio
    pin:
      number: GPIO33
      inverted: True
    id: gpio_33

light:
  - platform: monochromatic
    output: espCamLED
    name: esp-cam light
  - platform: binary
    output: gpio_33
    name: esp-cam led

  - platform: monochromatic
    output: gpio_13
    name: "Flash_light"
    id: flash_light
    on_turn_on:
      - delay: 60s
      - light.turn_off: flash_light

switch:
  - platform: restart
    name: esp-cam restart

binary_sensor:
  - platform: status
    name: esp-cam status
  - platform: fingerprint_grow
    id: fingerprint_enrolling
    name: "Fingerprint Enrolling"

  - platform: gpio
    pin:
      number: 12
      inverted: false
      mode:
        input: true
        pullup: true
    name: "RingButton"
    filters:
      - delayed_on: 10ms
    on_press:
      - dfplayer.play_mp3:
          file: 1
      - light.turn_on:
          id: flash_light
          brightness: 100%
      - delay: 30s     
      - light.turn_off: flash_light

uart:
  - id: uart_1
    tx_pin: GPIO3 
    rx_pin: GPIO1 
    baud_rate: 57600
  - id: uart_2
    tx_pin: GPIO14 
    rx_pin: GPIO15 
    baud_rate: 9600      

fingerprint_grow:
  uart_id: uart_1
  sensing_pin: GPIO2 #do not use pin GPIO16 or the cam will Crash
  on_finger_scan_matched:
    - homeassistant.event:
        event: esphome.test_node_finger_scan_matched
        data:
          finger_id: !lambda 'return finger_id;'
          confidence: !lambda 'return confidence;'

    - fingerprint_grow.aura_led_control:
        state: BREATHING
        speed: 200
        color: GREEN
        count: 1
    - text_sensor.template.publish:
        id: fingerprint_state
        state: "Autorizado"

  on_finger_scan_unmatched:
    - homeassistant.event:
        event: esphome.test_node_finger_scan_unmatched
    - fingerprint_grow.aura_led_control:
        state: FLASHING
        speed: 25
        color: RED
        count: 5
    - text_sensor.template.publish:
        id: fingerprint_state
        state: "Desconhecido"

  on_enrollment_scan:
    - homeassistant.event:
        event: esphome.test_node_enrollment_scan
        data:
          finger_id: !lambda 'return finger_id;'
          scan_num: !lambda 'return scan_num;'
    - fingerprint_grow.aura_led_control:
        state: FLASHING
        speed: 25
        color: BLUE
        count: 2
    - fingerprint_grow.aura_led_control:
        state: ALWAYS_ON
        speed: 0
        color: PURPLE
        count: 0
    - text_sensor.template.publish:
        id: fingerprint_state
        state: "Aprendendo"

  on_enrollment_done:
    - homeassistant.event:
        event: esphome.test_node_enrollment_done
        data:
          finger_id: !lambda 'return finger_id;'
    - fingerprint_grow.aura_led_control:
        state: BREATHING
        speed: 100
        color: BLUE
        count: 2
    - text_sensor.template.publish:
        id: fingerprint_state
        state: "Registrada"

  on_enrollment_failed:
    - homeassistant.event:
        event: esphome.test_node_enrollment_failed
        data:
          finger_id: !lambda 'return finger_id;'
    - fingerprint_grow.aura_led_control:
        state: FLASHING
        speed: 25
        color: RED
        count: 4
    - text_sensor.template.publish:
        id: fingerprint_state
        state: "Falha"

text_sensor:
  - platform: template
    id: fingerprint_state
    name: "Fingerprint State"
    lambda: |-
      return {"-"};
    update_interval: 6s

sensor:
  - platform: wifi_signal
    name: "espcam_wifi"
    update_interval: 30s
    
  - platform: fingerprint_grow
    fingerprint_count:
      name: "Fingerprint Count"
    last_finger_id:
      name: "Fingerprint Last Finger ID"
    last_confidence:
      name: "Fingerprint Last Confidence"
    status:
      name: "Fingerprint Status"
    capacity:
      name: "Fingerprint Capacity"
    security_level:
      name: "Fingerprint Security Level"

dfplayer:
  uart_id: uart_2
  on_finished_playback:
    then:
      logger.log: 'Playback finished'

```
Automations YAML:

# Final:


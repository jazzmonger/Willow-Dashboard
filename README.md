# Willow-Dashboard
Near realtime Dashboard for Willow WIS Inference server

![image](https://github.com/jazzmonger/Willow-Dashboard/assets/52110065/3864d898-d145-47e4-8a2c-228cd6fcaa6a)

This assumes WIS is operational and that the command ```nvidia-smi``` produces useable output. 

Install the SSH Integration
https://github.com/zhbjsh/homeassistant-ssh

make sure you can ssh to your willow server from a terminal window.

Once thats working, click CONFIGURE on the HA SSH integration (it took me forever to figure that out!) and paste these commands in the "Sensor commands*" area:

```
- command: >-
    docker logs  --tail 3 6c3f7e83827b 2>&1 |awk 'sub(/.*script\: */,""){f=1}
    f{if ( sub(/ *\[.*/,"") ) f=0; print}'
  scan_interval: 5
  timeout: 2
  sensors:
    - type: text
      maximum: 500
      name: STT Result
      key: STT_result
      icon: mdi:chart-line
- command: >-
    docker logs  --tail 2 6c3f7e83827b 2>&1 |grep -oP '(?<=took).*(?=ms)'| awk
    '{$1=$1};1'
  scan_interval: 5
  sensors:
    - type: number
      unit_of_measurement: ms
      name: STT Infer time
      key: STT_time
      icon: mdi:chart-line
- command: >-
    docker logs  --tail 1 6c3f7e83827b 2>&1 |awk '{ print substr($0,
    length($0)-2) }'
  scan_interval: 5
  timeout: 2
  sensors:
    - type: text
      maximum: 200
      name: STT Infer Speed up
      key: STT_speedup
      icon: mdi:chart-line
- command: >-
    nvidia-smi --query-gpu=utilization.gpu --format=csv,noheader  | awk '{ print
    substr( $0, 1, length($0)-2 ) }' | rev
  scan_interval: 1
  sensors:
    - type: number
      name: GPU Utilization
      key: gpu_utilization
      unit_of_measurement: "%"
      icon: mdi:chart-line
- command: >-
    nvidia-smi --query-gpu=memory.total --format=csv,noheader | awk '{ print
    substr( $0, 1, length($0)-4 ) }'
  scan_interval: 1
  sensors:
    - type: number
      name: GPU Mem. Total
      key: gpu_mem_total
      unit_of_measurement: MB
      icon: mdi:chart-line
- command: >-
    nvidia-smi --query-gpu=memory.used --format=csv,noheader | awk '{ print
    substr( $0, 1, length($0)-4 ) }'
  scan_interval: 1
  sensors:
    - type: number
      name: GPU Mem. Used
      key: gpu_mem_used
      unit_of_measurement: MB
      icon: mdi:chart-line
- command: >-
    nvidia-smi --query-gpu=memory.free --format=csv,noheader | awk '{ print
    substr( $0, 1, length($0)-4 ) }' 
  scan_interval: 1
  sensors:
    - type: number
      name: GPU Mem. Free
      key: gpu_mem_free
      unit_of_measurement: MB
      icon: mdi:chart-line
- command: "nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader  "
  scan_interval: 3
  sensors:
    - type: number
      name: GPU Temperature
      key: gpu_temp
      unit_of_measurement: C
      icon: mdi:chart-line
- command: nvidia-smi --query-gpu=pstate --format=csv,noheader | sed 's/^.\{1\}//'
  scan_interval: 1
  sensors:
    - type: text
      name: GPU Utilization Pstate
      key: gpu_pstate
      unit_of_measurement: P
      icon: mdi:chart-line
- command: /sbin/route -n | awk '/^0.0.0.0/ {print $NF}'
  sensors:
    - type: text
      name: Network interface
      key: network_interface
- command: cat /sys/class/net/&{network_interface}/address
  sensors:
    - type: text
      name: MAC address
      key: mac_address
- command: cat /sys/class/net/&{network_interface}/device/power/wakeup 2>/dev/null
  sensors:
    - type: binary
      name: Wake on LAN
      key: wake_on_lan
- command: uname -n
  sensors:
    - type: text
      name: Hostname
      key: hostname
- command: uname -m
  sensors:
    - type: text
      name: Machine type
      key: machine_type
- command: uname -s
  sensors:
    - type: text
      name: OS name
      key: os_name
- command: uname -r
  sensors:
    - type: text
      name: OS version
      key: os_version
- command: >-
    dmidecode | grep -A4 '^System Information' | awk -F": " '{if($0~/Product
    Name:/){a=$2} if($0~/Version:/){b=$2} if($0~/Manufacturer:/){c=$2}
    if($0~/Serial Number:/){d=$2}} END{print a; print b; print c; print d}'
  sensors:
    - type: text
      name: Device name
      key: device_name
    - type: text
      name: Device model
      key: device_model
    - type: text
      name: Manufacturer
      key: manufacturer
    - type: text
      name: Serial number
      key: serial_number
- command: >-
    cat /proc/cpuinfo | awk -F": " '{if($0~/^model name/){a=$2}
    if($0~/^processor/){b=$2} if($0~/^Hardware/){c=$2} if($0~/^Model/){d=$2}}
    END{print a; print b+1; print c; print d}' | sed -e "s/[[:space:]]\+/ /g"
  sensors:
    - type: text
      name: CPU name
      key: cpu_name
    - type: number
      name: CPU cores
      key: cpu_cores
    - type: text
      name: CPU hardware
      key: cpu_hardware
    - type: text
      name: CPU model
      key: cpu_model
- command: free -k | awk '/^Mem:/ {print $2}'
  sensors:
    - type: number
      name: Total memory
      key: total_memory
      unit_of_measurement: KiB
- command: free -k | awk '/^Mem:/ {print $4}'
  scan_interval: 30
  sensors:
    - type: number
      name: Free memory
      key: free_memory
      unit_of_measurement: KiB
- command: >-
    df -k | awk '/^\/dev\// {x=$4; $1=$2=$3=$4=$5=""; sub(/^ +/, "", $0); print
    $0 "|" x}'
  scan_interval: 60
  sensors:
    - type: number
      name: Free disk space
      key: free_disk_space
      dynamic: true
      separator: "|"
      unit_of_measurement: KiB
- command: top -bn1 | awk 'NR<4 && tolower($0) ~ /cpu/ {print 100-$8}'
  scan_interval: 30
  sensors:
    - type: number
      name: CPU load
      key: cpu_load
      unit_of_measurement: "%"
- command: >-
    for x in $(ls -d /sys/class/thermal/thermal_zone*); do echo $(cat
    $x/type),$(($(cat $x/temp)/1000)); done
  scan_interval: 60
  sensors:
    - type: number
      name: Temperature
      key: temperature
      dynamic: true
      separator: ","
      unit_of_measurement: °C
- command: ps -e | awk 'END {print NR-1}'
  scan_interval: 60
  sensors:
    - type: number
      name: Processes
      key: processes
```
For the graphing, I use a very cool integration called Plotly.  Highly configurable and flexible.
https://github.com/dbuezas/lovelace-plotly-graph-card

once you have that installed, paste this into a manual lovelace card's configuration. change to your liking!
```
type: custom:plotly-graph
hours_to_show: 0.1
raw_plotly_config: null
defaults:
  yaxes:
    side: left
    overlaying: 'y'
    visible: true
    showgrid: true
  entity:
    show_value: true
entities:
  - entity: sensor.willow_server_gpu_temp
    name: GPU Temperature
    yaxis: y1
    line:
      color: pink
      width: 1
    smoothing: 1
  - entity: sensor.willow_server_cpu_load
    name: CPU Load
    internal: false
    yaxis: y2
    line:
      color: white
    smoothing: 1
  - entity: sensor.willow_server_gpu_utilization
    name: GPU Load
    internal: false
    yaxis: y3
    line:
      color: yellow
    smoothing: 1
  - entity: sensor.willow_server_gpu_mem_used
    name: GPU Memory Used
    internal: false
    yaxis: y4
    line:
      color: aqua
    smoothing: 1
  - entity: sensor.willow_server_gpu_mem_free
    name: GPU Memory free
    internal: false
    yaxis: y5
    line:
      color: yellow
    smoothing: 1
  - entity: sensor.willow_server_gpu_pstate
    name: GPU P state
    internal: false
    yaxis: y6
    line:
      color: yellow
refresh_interval: auto
autorange_after_scroll: true
layout:
  height: 800
  yaxis:
    title: GPU Temp
    zeroline: false
    fixedrange: false
  yaxis2:
    fixedrange: true
    zeroline: false
    title: CPU Load
  yaxis3:
    title: GPU Load
    zeroline: true
    fixedrange: false
    range:
      - 0
      - 100
  yaxis4:
    title: GPU Mem Used
    zeroline: false
    fixedrange: false
  yaxis5:
    title: GPU Mem free
    zeroline: false
    fixedrange: false
  yaxis6:
    title: GPU P-State
    zeroline: false
    fixedrange: false
    range:
      - 0
      - 11
  grid:
    ygap: 0.2
    rows: 6
    columns: 1
    pattern: coupled
    roworder: top to bottom
  xaxis:
    rangeselector:
      bgcolor: '#474747'
      'y': 1.14
      buttons:
        - count: 1
          step: minute
        - count: 10
          step: minute
        - count: 30
          step: minute
        - count: 1
          step: hour
        - count: 4
          step: hour
```

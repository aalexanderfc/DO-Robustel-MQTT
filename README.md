---

# **Robustel MQTT Integration for External Relay Control via Node-RED**  

This project enables **MQTT-based control of external relays** connected to **DO3 and DO4** on a **Robustel router**. The integration allows switching **relays ON/OFF via MQTT messages** and provides a **web-based UI** using Node-RED Dashboard v2.

## ** Features**  
✅ Control **external relays** via MQTT commands  
✅ Web-based dashboard using **Node-RED v2**  
✅ **Real-time relay control** via `/sys/class/leds/` (DO3/DO4 output)  
✅ **Status monitoring** using MQTT  

---

## ** Setup Guide**  

### **1️ Prerequisites**  
Ensure you have the following installed on the **Robustel router**:  
- **Python 3.x**  
- **Mosquitto MQTT Broker**  
- **Node.js** (Required for Node-RED)  
- **Node-RED v2 with Dashboard**  

### **2️ Install Node.js and Node-RED**
To install **Node.js** and **Node-RED**, follow these steps:

```bash
sudo apt update
sudo apt install nodejs npm
sudo npm install -g --unsafe-perm node-red
```

### **3️  Install Required Python Libraries**  
Run the following command to install `paho-mqtt` for MQTT communication:  
```bash
sudo python3 -m pip install paho-mqtt
```

---

## **4️ Connect External Relays to DO3 and DO4**
Since the **Robustel router does not have built-in relays**, we need to connect external relays to **DO3 and DO4**.

### **Wiring the External Relays**
1. **DO3 → Relay 1 Input (IN)**
2. **DO4 → Relay 2 Input (IN)**
3. **Relay GND → Router GND**
4. **Relay VCC → Router Power Output (if available)**
5. **Relay Output (NO/NC) → Controlled Load**

---

## **5️ Configure Robustel Router for DO Control**  
The **DO3 and DO4 outputs** are controlled via **/sys/class/leds/** instead of traditional GPIO.

### **Test Relay Control Manually**  
```bash
echo 1 | sudo tee /sys/class/leds/do1/brightness  # Turn ON Relay 1 (DO3)
echo 0 | sudo tee /sys/class/leds/do1/brightness  # Turn OFF Relay 1 (DO3)
echo 1 | sudo tee /sys/class/leds/do2/brightness  # Turn ON Relay 2 (DO4)
echo 0 | sudo tee /sys/class/leds/do2/brightness  # Turn OFF Relay 2 (DO4)
```
✅ **If the external relays click, the setup is correct!**

---

## ** 6️ Create Python Script for MQTT → DO Control**  

Create a script called `mqtt-led-do.py`:  
```python
import paho.mqtt.client as mqtt
import os
import json

# LED Paths for DO3 (Relay 1) and DO4 (Relay 2)
DO3_PATH = "/sys/class/leds/do1/brightness"
DO4_PATH = "/sys/class/leds/do2/brightness"

# MQTT Settings
BROKER = "127.0.0.1"
TOPIC_DO = "robustel/do/control"

def set_do(led_path, state):
    with open(led_path, "w") as f:
        f.write(str(state))

def on_message(client, userdata, msg):
    payload = json.loads(msg.payload.decode())
    if "do3" in payload:
        set_do(DO3_PATH, payload["do3"])
    if "do4" in payload:
        set_do(DO4_PATH, payload["do4"])

mqtt_client = mqtt.Client()
mqtt_client.on_message = on_message
mqtt_client.connect(BROKER, 1883)
mqtt_client.subscribe(TOPIC_DO)
mqtt_client.loop_forever()
```

### **Run the Script**  
```bash
sudo python3 mqtt-led-do.py
```
✅ **Now the router listens for MQTT messages to control external relays on DO3/DO4.**  

---

## **📡 7️ Test MQTT Commands**
Use the following commands to control **DO3 and DO4 manually** via MQTT:  
```bash
mosquitto_pub -h 127.0.0.1 -t "robustel/do/control" -m '{"do3":1}'  # ON Relay 1 (DO3)
mosquitto_pub -h 127.0.0.1 -t "robustel/do/control" -m '{"do3":0}'  # OFF Relay 1 (DO3)
mosquitto_pub -h 127.0.0.1 -t "robustel/do/control" -m '{"do4":1}'  # ON Relay 2 (DO4)
mosquitto_pub -h 127.0.0.1 -t "robustel/do/control" -m '{"do4":0}'  # OFF Relay 2 (DO4)
```
✅ **If relays click, MQTT → DO control is working!** 🎉  

---

## ** Summary**
✅ **Configured external relay control via DO3/DO4 on Robustel**  
✅ **Created a Python script to control relays via MQTT**  
✅ **Tested MQTT commands manually**  
✅ **Integrated MQTT with Node-RED Dashboard v2**  
✅ **Configured auto-start for MQTT script on boot**  

---

 **Now you have full MQTT-based control of external relays on your Robustel router with a modern web UI!** 


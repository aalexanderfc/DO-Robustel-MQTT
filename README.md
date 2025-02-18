---

# **Robustel MQTT Integration for DO Control via Node-RED**  

This project enables MQTT-based **control of DO3 and DO4 relays** on a **Robustel router** using **Node-RED Dashboard v2**. The integration allows switching **relays ON/OFF via MQTT messages** and provides a **web-based UI** using Node-RED.

## **📌 Features**  
✅ Control DO3 and DO4 via MQTT commands  
✅ Web-based dashboard using **Node-RED v2**  
✅ **Real-time relay control** via `/sys/class/leds/`  
✅ **Status monitoring** using MQTT  

---

## **🚀 Setup Guide**  

### **1️⃣ Prerequisites**  
Ensure you have the following installed on the **Robustel router**:  
- **Python 3.x**  
- **Mosquitto MQTT Broker**  
- **Node-RED v2 with Dashboard**  

### **2️⃣ Install Required Python Libraries**  
Run the following command to install `paho-mqtt` for MQTT communication:  
```bash
sudo python3 -m pip install paho-mqtt
```

---

## **🔧 3️⃣ Configure Robustel Router for DO Control**  
The **DO3 and DO4 relays** are controlled via **/sys/class/leds/** instead of traditional GPIO.

### **Test Relay Control Manually**  
```bash
echo 1 | sudo tee /sys/class/leds/do1/brightness  # Turn ON DO3
echo 0 | sudo tee /sys/class/leds/do1/brightness  # Turn OFF DO3
echo 1 | sudo tee /sys/class/leds/do2/brightness  # Turn ON DO4
echo 0 | sudo tee /sys/class/leds/do2/brightness  # Turn OFF DO4
```
✅ **If relays click, the setup is correct!**

---

## **📝 4️⃣ Create Python Script for MQTT → DO Control**  

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
✅ **Now the router listens for MQTT messages to control DO3/DO4.**  

---

## **📡 5️⃣ Test MQTT Commands**
Use the following commands to control **DO3 and DO4 manually** via MQTT:  
```bash
mosquitto_pub -h 127.0.0.1 -t "robustel/do/control" -m '{"do3":1}'  # ON DO3
mosquitto_pub -h 127.0.0.1 -t "robustel/do/control" -m '{"do3":0}'  # OFF DO3
mosquitto_pub -h 127.0.0.1 -t "robustel/do/control" -m '{"do4":1}'  # ON DO4
mosquitto_pub -h 127.0.0.1 -t "robustel/do/control" -m '{"do4":0}'  # OFF DO4
```
✅ **If relays click, MQTT → DO control is working!** 🎉  

---

## **🌍 6️⃣ Configure Node-RED Dashboard v2**
### **A. Open Node-RED**
Start Node-RED:
```bash
node-red
```
Access UI:
```
http://<ROUTER_IP>:1880
```

### **B. Install MQTT & Dashboard Nodes**
1. **Go to `Manage Palette` → `Install`**  
2. Install:  
   ```
   node-red-node-mqtt
   node-red-dashboard
   ```

### **C. Add Switch for DO3 and DO4**
1. Drag **`dashboard → switch (v2)`** into the flow.  
2. **Double-click the switch node** and set:
   - **Name**: `DO3 Control`
   - **Topic**: `robustel/do/control`
   - **Payload ON**: `{"do3":1}`
   - **Payload OFF**: `{"do3":0}`
   - **Enable JSON Output** ✅  

3. **Duplicate the switch for DO4** and set:
   - **Name**: `DO4 Control`
   - **Payload ON**: `{"do4":1}`
   - **Payload OFF**: `{"do4":0}`  

### **D. Connect to `MQTT Out`**
1. **Drag an `MQTT out` node** into the flow.
2. **Set MQTT broker**: `127.0.0.1`
3. **Connect switches to `MQTT out`**
4. **Deploy the flow**

### **E. Open Web UI & Test**
Open:
```
http://<ROUTER_IP>:1880/ui
```
✅ **Toggle switches to control relays in real-time!** 🎨  

---

## **🛠 7️⃣ Make MQTT Script Run at Startup**
To ensure `mqtt-led-do.py` starts automatically on reboot:

1. **Create a systemd service file**  
```bash
sudo nano /etc/systemd/system/mqtt-led.service
```
2. **Paste the following configuration**:
```ini
[Unit]
Description=MQTT to DO Control Service
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/username/mqtt-led-do.py
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```
3. **Enable the service**:
```bash
sudo systemctl daemon-reload
sudo systemctl enable mqtt-led
sudo systemctl start mqtt-led
```
✅ **Now, the script runs automatically on boot!** 🔥  

---

## **📌 Summary**
✅ **Configured DO3/DO4 relay control via `/sys/class/leds/`**  
✅ **Created a Python script to control relays via MQTT**  
✅ **Tested MQTT commands manually**  
✅ **Integrated MQTT with Node-RED Dashboard v2**  
✅ **Configured auto-start for MQTT script on boot**  

---

🚀 **Now you have full MQTT-based control of your Robustel router relays with a modern web UI!** 🎉🔥  

---


# Smart-Factory
Fischertechnik factory Mongodb integration

Fischertechnik GitHub repo: <https://github.com/fischertechnik/txt_training_factory>

## Fischertechnik Factory Initial Setup

Official Manual:
<https://www.fischertechnik.de/-/media/fischertechnik/fite/service/elearning/lehren/lernfabrik/fabrik_2019_englisch_neu.ashx>

1. Reset TP-Link router by pushing the reset button for ~5sec
2. Connect to the TP-Link WIFI and open <http://tplinkwifi.net/>. Password on the TP-Link router.
3. Login with admin:admin
4. Switch "Operation Mode" to WISP
![WISP Configuration](/doc/images/TP-Link-OperationMode.png)
5. Configure "Wireless" settings to connect to your regular (internet connectivity) WLAN
![WIFI Configuration](/doc/images/TP-Link-WIFISetup.png)
6. Add port forwarding for SSH and the MQTT broker for direct access without switching WLAN
![Port Forwarding](/doc/images/TP-Link-PortForward.png)

## Remote MQTT Broker Configuration

The Fischertechnik factory has MQTT broker deployed on their TXT controllers with one main controller, where we will configure the bridge. This MQTT broker is considered the remote broker which will receive/send messages from/to the main TXT MQTT controller.

The broker is started with a custom config file in the mqtt folder. 
You have to run the command from within the mqtt folder!

```
docker run -it -p 1883:1883 -p 9001:9001 -v $(pwd)/mosquitto.conf:/mosquitto/config/mosquitto.conf eclipse-mosquitto
```

## TXT MQTT Bridge Configuration

The factory contains 6 TXT controllers which communicate within eachother via MQTT. The SSC TXT controller is the main/central controller. The MQTT broker on this controller will be configured to forward/receive messages to the previously configured remote MQTT broker via the following MQTT bridge configuration.

Changing this configuration will disconnect the factory from the Fischercloud!  
(TODO: Check if both configurations can be run in parallel)

The MQTT interface of the main controller is documented here:  
<https://github.com/fischertechnik/txt_training_factory/blob/master/TxtSmartFactoryLib/doc/MqttInterface.md>

The Mosquitto config file documentation is available here:  
<https://mosquitto.org/man/mosquitto-conf-5.html>

```
connection ft-txt-bridge-cloud
address 192.168.1.113:1883
#bridge_capath /etc/ssl/certs
notifications false
cleansession false #on connection dropping
#remote_username 6875
#remote_password 4!4?P4mTVop6bZ
local_username txt
local_password xtx
topic i/# both 1 "" /j1/txt/6875/
topic o/# both 1 "" /j1/txt/6875/
topic c/# both 1 "" /j1/txt/6875/
topic f/# both 1 "" /j1/txt/6875/
try_private false
bridge_attempt_unsubscribe false
```

## Node-RED Dashboard

<https://github.com/fischertechnik/txt_training_factory/tree/master/Node-RED>

```
docker run -it -p 1880:1880 -v node_red_data:/data --name mynodered nodered/node-red
```

1. Access the dashboard <http://127.0.0.1:1880/>
2. Install "node-red-dashboards" -> "Menu"/"Manage palette"
![Node-Red-Dashboards](/doc/images/node-red-dashboard.png)
3. Import "flows.json"
3. Verify/change MQTT broker IP address "Menu"/"Configuration nodes"
4. Deploy and open dashboard <http://127.0.0.1:1880/ui/>
5. If the camera doesn't sent pictures, go to the "Lernfabrik4.0 - conf" flow and hit the activate camera button

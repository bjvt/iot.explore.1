https://www.thethingsnetwork.org/forum/t/how-can-i-forward-the-messages-from-ttn-to-mqtt-server-is-there-any-option-for-mqtt-integration/29060
---------- this is the exercise I'm following with the TTN, PI4, MQTTX
mqtt key on TTN generated 10/24@23:03 CST,  641-1306-1-ND
NNSXS.BN3CPH3WSKPBYBMN74PL44V7IL32S2SLQ4KCWSA.XWT5QIQLZLPKJHYDMKIAXVRZD6ANP3D35T5Q2UVMPXPE36W235RQ


---- this is the bash script on the PI4	
pi@p0:~ $ cat bash.time.sh 
#!/bin/bash

    export PATH=/usr/local/bin:$PATH

    counter=0
    #rak811v3 hard-reset
    #sleep 15
    rak811v3 join > /home/pi.log 2>&1
    sleep 10
while true; do
    current_time=$(date +%Y%m%d%H%M%S)
    rak811v3 send  $current_time
    rak811v3 send hi

    counter=$((counter + 1))
    echo "Loop has been called $counter times."
    rak811v3 send $counter
    sleep 1800  # Sleep for 15 minutes

done
# sudo systemctl start bash.time.service
# sudo journalctl -u bash.service
# sudo systemctl status  bash.service


---------- this is the service unit file ------------/etc/systemd/system.bash.service -755- permission -----------
pi@p0:/etc/systemd/system $ cat bash.service
[Unit]
Description=My Bash Script

[Service]
Type=simple
ExecStart=/home/pi/bash.time.sh

[Install]
WantedBy=multi-user.target

-------- MQTT Server ------------
username - iot-barry@ttn
password is the API key
Client ID - mqttx_ab20b365
MQTT version 3.1.1
Host nam1.cloud.thethings.network
topic v3/iot-barry@ttn/devices/eui-ac1f09fffe03dd5f/#
end device - eui-ac1f09fffe03dd5f

----- nodered on PI4 --------
pi@192.168.1.185, nodered is port 1880
https://www.emqx.com/en/blog/using-node-red-to-process-mqtt-data
https://learn.pi-supply.com/make/getting-started-with-the-raspberry-pi-lora-node-phat/
used this from the node-red site - bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)

// Subscribe to MQTT topic
[MQTT In]
topic: "my-topic"

// Save to file
[File Out]
filename: "my-file.csv"
format: "csv"
fields: "timestamp,value"

// Determine value
[Function]
function(msg) {
  // Get the value from the MQTT message
  var value = msg.payload.value;

  // Determine if the value is over the threshold
  var threshold = 100;
  var isOverThreshold = value > threshold;

  // Set the output message
  msg.payload = {
    value: value,
    isOverThreshold: isOverThreshold
  };

  return msg;
}

// Publish MQTT if over value threshold
[MQTT Out]
topic: "my-topic-threshold"
qos: 1
retain: true
payload: msg.payload.isOverThreshold


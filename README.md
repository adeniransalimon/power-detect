<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Power Status Monitor</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.min.js"></script>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            background-color: #222;
            color: white;
            font-family: Arial, sans-serif;
            margin: 0;
        }
        .bulb {
            width: 100px;
            height: 100px;
            background-color: #555;
            border-radius: 50%;
            margin-bottom: 20px;
            transition: all 0.3s ease;
            box-shadow: 0 0 10px #000 inset;
        }
        /* Classes applied dynamically via JavaScript */
        .bulb.on {
            background-color: #ffeb3b;
            box-shadow: 0 0 50px #ffeb3b, 0 0 100px #ffeb3b;
        }
        .bulb.off {
            background-color: #333;
            box-shadow: none;
        }
        h1 {
            font-size: 2.5rem;
            margin: 0;
        }
        #statusText {
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div id="lightBulb" class="bulb off"></div>
    <h1>Status: <span id="statusText">Connecting...</span></h1>
    <script>
        // --- HiveMQ WebSockets Configuration ---
        const mqttServer = "082ee80754e24521b3c0e901a1ac9c31.s1.eu.hivemq.cloud";
        const mqttPort = 8883; // FIX: Port 443 bypasses network firewalls on secure WebSockets(WSS)
        const mqttUser = "ESP32_power_detect";
        const mqttPass = "gp2powerDetect";
        const topic = "home/power/status";

        // Generate random client ID for the browser
        const clientId = "WebClient-" + Math.random().toString(16).substr(2, 8);

        // Initialize MQTT Client with standard path structure
        const client = new Paho.MQTT.Client(mqttServer, mqttPort, "/mqtt", clientId);

        // Set callback handlers
        client.onConnectionLost = onConnectionLost;
        client.onMessageArrived = onMessageArrived;

        // Connect options with SSL enabled
        const options = {
            useSSL: true,
            timeout: 10,
            keepAliveInterval: 60,
            userName: mqttUser,
            password: mqttPass,
            onSuccess: onConnect,
            onFailure: onFailure
        };

        // Attempt connection
        client.connect(options);

        function onConnect() {
            console.log("Connected to HiveMQ Cloud");
            document.getElementById("statusText").innerText = "Waiting for data...";
            document.getElementById("statusText").style.color = "#ff9800"; // Standby Orange
            client.subscribe(topic);
        }

        function onFailure(message) {
            console.log("Connection failed: " + message.errorMessage);
            document.getElementById("statusText").innerText = "Connection Failed";
            document.getElementById("statusText").style.color = "#f44336"; // Error Red
        }

        function onConnectionLost(responseObject) {
            if (responseObject.errorCode !== 0) {
                console.log("Connection lost: " + responseObject.errorMessage);
                document.getElementById("statusText").innerText = "Disconnected";
                document.getElementById("statusText").style.color = "#f44336";
                document.getElementById("lightBulb").className = "bulb off";
            }
        }

        function onMessageArrived(message) {
            const payload = message.payloadString.trim();
            console.log("Message arrived: " + payload);
            const bulb = document.getElementById("lightBulb");
            const statusText = document.getElementById("statusText");

            if (payload === "ON") {
                bulb.className = "bulb on";
                statusText.innerText = "Light Active";
                statusText.style.color = "#ffeb3b";
            } else if (payload === "OFF") {
                bulb.className = "bulb off";
                statusText.innerText = "No Light";
                statusText.style.color = "#aaa";
            }
        }
    </script>
</body>
</html>

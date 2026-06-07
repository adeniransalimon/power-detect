<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Power Status Display Panel</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.min.js"></script>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #121212;
            color: #ffffff;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .panel {
            background: #1e1e1e;
            padding: 40px;
            border-radius: 20px;
            text-align: center;
            box-shadow: 0 10px 30px rgba(0,0,0,0.7);
            border: 1px solid #2d2d2d;
        }
        .bulb {
            font-size: 100px;
            margin: 30px 0;
            transition: all 0.3s ease;
            display: inline-block;
        }
        /* Dynamic indicator rendering configurations */
        .bulb.glowing {
            text-shadow: 0 0 50px #ffeb3b, 0 0 100px #ffeb3b;
            transform: scale(1.1);
            opacity: 1.0;
        }
        .bulb.dark {
            opacity: 0.15;
            text-shadow: none;
            transform: scale(1.0);
        }
        #status-msg {
            font-size: 1.6rem;
            font-weight: bold;
            letter-spacing: 1px;
            margin-top: 10px;
            transition: color 0.3s ease;
        }
    </style>
</head>
<body>

    <div class="panel">
        <h2>Mains Supply Indicator</h2>
        <div id="lightBulb" class="bulb dark">💡</div>
        <div id="status-msg">Connecting to Server...</div>
    </div>

    <script>
        // --- HiveMQ WebSockets Production Tunneling Declarations ---
        const mqttServer = "03296b8eda5d43dfbdd836411b8b6c18.s1.eu.hivemq.cloud";
        const mqttPort   = 8884; // Standard cloud broker WebSockets SSL Port
        const mqttUser   = "power_detect";
        const mqttPass   = "gp2powerDetect";
        const topic      = "home/power/status";

        // Generate unique client signature token to allow parallel device viewing
        const clientId = "WebClient-" + Math.random().toString(16).substr(2, 8);
        
        // FIX: The path must be empty "" for HiveMQ Cloud server handshakes to succeed
        const client = new Paho.MQTT.Client(mqttServer, mqttPort, "", clientId);

        client.onConnectionLost = onConnectionLost;
        client.onMessageArrived = onMessageArrived;

        const options = {
            useSSL: true,
            timeout: 3,
            keepAliveInterval: 60,
            userName: mqttUser,
            password: mqttPass,
            onSuccess: onConnect,
            onFailure: onFailure
        };

        // Open non-blocking socket link 
        client.connect(options);

        function onConnect() {
            console.log("Securely bridged with HiveMQ WebSockets");
            document.getElementById("status-msg").innerText = "WAITING FOR DATA...";
            document.getElementById("status-msg").style.color = "#ff9800"; // Orange standby text
            client.subscribe(topic);
        }

        function onFailure(message) {
            console.log("Connection parameter handshake failed: " + message.errorMessage);
            document.getElementById("status-msg").innerText = "CONNECTION FAILED";
            document.getElementById("status-msg").style.color = "#f44336"; 
        }

        function onConnectionLost(responseObject) {
            if (responseObject.errorCode !== 0) {
                console.log("Disconnected: " + responseObject.errorMessage);
                document.getElementById("status-msg").innerText = "OFFLINE (SERVER LOST)";
                document.getElementById("status-msg").style.color = "#f44336";
                const bulb = document.getElementById("lightBulb");
                bulb.className = "bulb dark";
            }
        }

        function onMessageArrived(message) {
            const payload = message.payloadString.trim();
            console.log("Payload packet ingested: " + payload);
            const bulb = document.getElementById("lightBulb");
            const text = document.getElementById("status-msg");

            if (payload === "ON") {
                bulb.className = "bulb glowing";
                text.innerText = "LIGHT IS ON";
                text.style.color = "#4caf50"; // Clean high-visibility dashboard green
            } else if (payload === "OFF") {
                bulb.className = "bulb dark";
                text.innerText = "LIGHT IS OFF";
                text.style.color = "#f44336"; // Deep alerting danger panel red
            }
        }
    </script>
</body>
</html>

## Docker container for Raspberry Pi Sensehat and USB camera

Testing the programs separately. The Camera works with pygame on Ubuntu, but on CentOS 8, I was not able to install the dependencies for pygame. Therefore, run the testcam.py in podman/docker container.
```
python temperature.py
python testcam.py
python sparkles.py
```

Build and push the image
```
docker build -t karve/sensehat .
docker run --privileged --name sensehat -ti karve/sensehat bash
docker push karve/sensehat
```

This deployment in microshift will start sending images and temperature readings from SenseHat to Node-Red
```
kubectl create -f sensehat.yaml
```

Delete the deployment to stop sending to Node Red
```
kubectl delete -f sensehat.yaml
```

## Node-Red to upload and display the image along with SenseHat readings
1. Create a IBM Cloud free tier account at https://www.ibm.com/cloud/free and login to Console (top right).
2. Create an API Key and save it, Manage->Access->IAM->API Key->Create an IBM Cloud API Key
2. Click on Catalog and Search for "Node-Red App", select it and click on "Get Started"
3. Give a unique App name For example xxxxx-node-red and select the region nearest to you
4. Select the Pricing Plan Lite, if you already have an existing instance of Cloudant, you may select it in Pricing Plan
5. Click Create
6. Under Deployment Automation -> Configure Continuous Delivery, click on "Deploy your app"
7. Select the deployment target "Cloud Foundry"
8. Enter the IBM Cloud API Key from Step 2, or click on "New" to create one
9. The rest of the fields Region, Organization, Space will automatically get filled up. Use the default 256MB Memory and click "Next"
10. In "Configure the DevOps toolchain", click Create
11. Wait for 10 minutes for the Node Red instance to start
12. Click on the "Visit App URL"
13. On the Node Red page, create a new userid and password
14. In Manage Palette, install the node-red-contrib-image-tools, node-red-contrib-image-output, node-red-node-base64, node-red-node-random and node-red-contrib-aedes
15. Import required Chat and Image Viewer flows as below.

### Picture on Node Red
Connect to your own instance of NodeRed on IBM Cloud http://mynodered.mybluemix.net/simple
```
[
    {
        "id": "7c30fb48.99cd2c",
        "type": "http in",
        "z": "4024e2467929a360",
        "name": "",
        "url": "/simple",
        "method": "get",
        "upload": false,
        "swaggerDoc": "",
        "x": 170,
        "y": 140,
        "wires": [
            [
                "3153fbd3.203a64"
            ]
        ]
    },
    {
        "id": "3153fbd3.203a64",
        "type": "template",
        "z": "4024e2467929a360",
        "name": "html",
        "field": "payload",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "<h1>Upload a file here:</h1>\n\n<form action=\"/upload\" method=\"POST\" enctype=\"multipart/form-data\">\n    <input type=\"file\" name=\"myFile\" />\n    <input type=\"submit\" value=\"Submit\">\n</form>",
        "output": "str",
        "x": 510,
        "y": 140,
        "wires": [
            [
                "737e44d2.373a64"
            ]
        ]
    },
    {
        "id": "737e44d2.373a64",
        "type": "http response",
        "z": "4024e2467929a360",
        "name": "",
        "x": 650,
        "y": 140,
        "wires": []
    },
    {
        "id": "46e01eea.4a46a",
        "type": "comment",
        "z": "4024e2467929a360",
        "name": "Simple",
        "info": "",
        "x": 150,
        "y": 100,
        "wires": []
    },
    {
        "id": "7d8e179a.283e4",
        "type": "http response",
        "z": "4024e2467929a360",
        "name": "",
        "x": 650,
        "y": 200,
        "wires": []
    },
    {
        "id": "25859a94.7d9976",
        "type": "http in",
        "z": "4024e2467929a360",
        "name": "",
        "url": "/upload",
        "method": "post",
        "upload": true,
        "swaggerDoc": "",
        "x": 170,
        "y": 200,
        "wires": [
            [
                "d7e5df8b.97a4e8",
                "d63bc821ef4d1e2e"
            ]
        ]
    },
    {
        "id": "e1643981.20d7c8",
        "type": "template",
        "z": "4024e2467929a360",
        "name": "text",
        "field": "payload",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "File {{name}} uploaded\n",
        "output": "str",
        "x": 510,
        "y": 200,
        "wires": [
            [
                "7d8e179a.283e4"
            ]
        ]
    },
    {
        "id": "d7e5df8b.97a4e8",
        "type": "function",
        "z": "4024e2467929a360",
        "name": "toBase64",
        "func": "msg.name = msg.req.files[0].originalname;\n\nif (msg.req.files[0].mimetype.includes('image')) {\n    msg.payload = `<img src=\"data:image/gif;base64,${msg.req.files[0].buffer.toString('base64')}\">`;\n} else {\n    msg.payload = msg.req.files[0].buffer.toString();\n}\n\nreturn msg;",
        "outputs": 1,
        "noerr": 0,
        "x": 360,
        "y": 200,
        "wires": [
            [
                "e1643981.20d7c8"
            ]
        ]
    },
    {
        "id": "d63bc821ef4d1e2e",
        "type": "function",
        "z": "4024e2467929a360",
        "name": "toBase64",
        "func": "msg.name = msg.req.files[0].originalname;\n\nif (msg.req.files[0].mimetype.includes('image')) {\n    msg.payload = `${msg.req.files[0].buffer.toString('base64')}`;\n} else {\n    msg.payload = msg.req.files[0].buffer.toString();\n}\n\nreturn msg;",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 350,
        "y": 240,
        "wires": [
            [
                "7518e9142f5680d8",
                "43fafba17925df91"
            ]
        ]
    },
    {
        "id": "7518e9142f5680d8",
        "type": "image viewer",
        "z": "4024e2467929a360",
        "name": "",
        "width": "640",
        "data": "payload",
        "dataType": "msg",
        "active": false,
        "x": 510,
        "y": 240,
        "wires": [
            []
        ]
    },
    {
        "id": "1880286475e54335",
        "type": "template",
        "z": "4024e2467929a360",
        "name": "html",
        "field": "payload",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "<p></p>File {{name}} uploaded !</p>\n\n<h2>Contents:</h2>\n\n<pre>\n{{{payload}}}\n</pre>",
        "output": "str",
        "x": 800,
        "y": 200,
        "wires": [
            []
        ]
    },
    {
        "id": "43fafba17925df91",
        "type": "image",
        "z": "4024e2467929a360",
        "name": "",
        "width": "640",
        "data": "payload",
        "dataType": "msg",
        "thumbnail": false,
        "active": true,
        "pass": false,
        "outputs": 0,
        "x": 180,
        "y": 280,
        "wires": []
    },
    {
        "id": "0ea52358ab8e2391",
        "type": "comment",
        "z": "4024e2467929a360",
        "name": "Replace-text-by-html",
        "info": "",
        "x": 960,
        "y": 200,
        "wires": []
    }
]
```

### Chat Server on Node Red
Connect to your own instance of NodeRed on IBM Cloud https://mynodered.mybluemix.net/chat
```
[
    {
        "id": "1f044347b4962306",
        "type": "websocket in",
        "z": "6e675b81333ad533",
        "name": "",
        "server": "bc740d23.438bf",
        "x": 130,
        "y": 60,
        "wires": [
            [
                "09dc90bbddd6b23d"
            ]
        ]
    },
    {
        "id": "09dc90bbddd6b23d",
        "type": "function",
        "z": "6e675b81333ad533",
        "name": "",
        "func": "delete msg._session;\nreturn msg;\n\n",
        "outputs": 1,
        "x": 304,
        "y": 60,
        "wires": [
            [
                "f16a1212f8341911"
            ]
        ]
    },
    {
        "id": "f16a1212f8341911",
        "type": "websocket out",
        "z": "6e675b81333ad533",
        "name": "",
        "server": "bc740d23.438bf",
        "x": 485,
        "y": 60,
        "wires": []
    },
    {
        "id": "428922193679ca2d",
        "type": "http in",
        "z": "6e675b81333ad533",
        "name": "",
        "url": "/chat",
        "method": "get",
        "x": 138,
        "y": 128,
        "wires": [
            [
                "316414fb7c1e99f9"
            ]
        ]
    },
    {
        "id": "316414fb7c1e99f9",
        "type": "template",
        "z": "6e675b81333ad533",
        "name": "",
        "field": "payload",
        "fieldType": "msg",
        "syntax": "mustache",
        "template": "<head>\n  <meta name=\"viewport\" content=\"width=320, initial-scale=1\">\n  <title>Chat</title>\n<script type=\"text/javascript\">\n\n  function createSystemMessage(message) {\n    var message = document.createTextNode(message);\n\n    var messageBox = document.createElement('p');\n    messageBox.className = 'system';\n\n    messageBox.appendChild(message);\n\n    var chat = document.getElementById('chat_box');\n    chat.appendChild(messageBox);\n  }\n\n  function createUserMessage(user, message) {\n    var user = document.createTextNode(user + ': ');\n\n    var userBox = document.createElement('span');\n    userBox.className = 'username';\n    userBox.appendChild(user);\n\n    var message = document.createTextNode(message);\n\n    var messageBox = document.createElement('p');\n    messageBox.appendChild(userBox);\n    messageBox.appendChild(message);\n\n    var chat = document.getElementById('chat_box');\n    chat.appendChild(messageBox);\n  }\n\n  var wsUri = \"ws://{{req.headers.host}}/ws/chat\";\n  var wsglobal;\n  \n  function connect() {\n    var ws = new WebSocket(wsUri);\n\n    ws.onopen = function(ev) {\n      createSystemMessage('[Connected]');\n    };\n\n    ws.onclose = function(ev) {\n      createSystemMessage('[Disconnected]');\n      connect();\n    }\n\n    ws.onmessage = function(ev) {\n      var payload = JSON.parse(ev.data);\n      createUserMessage(payload.user, payload.message);\n\n      var chat = document.getElementById('chat_box');\n      chat.scrollTop = chat.scrollHeight;\n    }\n    \n    wsglobal=ws;\n  }\n  \n  connect();\n  function clearMessages() {\n      document.getElementById('chat_box').innerHTML=\"\";\n  }\n  function sendMessage() {\n    var user = document.getElementById('user');\n    var message = document.getElementById('message');\n\n    var payload = {\n      message: message.value,\n      user: user.value,\n      ts: (new Date()).getTime()\n    };\n\n    wsglobal.send(JSON.stringify(payload));\n    message.value = \"\";\n  };\n</script>\n\n<style type=\"text/css\">\n  * {\n    font-family: \"Palatino Linotype\", \"Book Antiqua\", Palatino, serif;\n    font-style: italic;\n    font-size: 24px;\n  }\n\n  html, body, #wrapper {\n    margin: 0;\n    padding: 0;\n    height: 100%;\n  }\n\n  #wrapper {\n    background-color: #ecf0f1;\n  }\n\n  #chat_box {\n    box-sizing: border-box;\n    height: 100%;\n    overflow: auto;\n    padding-bottom: 50px;\n  }\n\n  #footer {\n    box-sizing: border-box;\n    position: fixed;\n    bottom: 0;\n    height: 50px;\n    width: 100%;\n    background-color: #2980b9;\n  }\n\n  #footer .content {\n    padding-top: 4px;\n    position: relative;\n  }\n\n  #user { width: 15%; }\n  #message { width: 58%; }\n  #clear_btn {\n    width: 10%;\n    position: absolute;\n    right: 15%;\n    bottom: 0;\n    margin: 0;\n  }\n  #send_btn {\n    width: 10%;\n    position: absolute;\n    right: 0;\n    bottom: 0;\n    margin: 0;\n  }\n\n  .content {\n    width: 70%;\n    margin: 0 auto;\n  }\n\n  input[type=\"text\"],\n  input[type=\"button\"] {\n    border: 0;\n    color: #fff;\n  }\n\n  input[type=\"text\"] {\n    background-color: #146EA8;\n    padding: 3px 10px;\n  }\n\n  input[type=\"button\"] {\n    background-color: #f39c12;\n    border-right: 2px solid #e67e22;\n    border-bottom: 2px solid #e67e22;\n    min-width: 70px;\n    display: inline-block;\n  }\n\n  input[type=\"button\"]:hover {\n    background-color: #e67e22;\n    border-right: 2px solid #f39c12;\n    border-bottom: 2px solid #f39c12;\n    cursor: pointer;\n  }\n\n  .system,\n  .username {\n    color: #aaa;\n    font-style: italic;\n    font-family: monospace;\n    font-size: 16px;\n  }\n\n  @media(max-width: 1000px) {\n    .content { width: 90%; }\n  }\n\n  @media(max-width: 780px) {\n    #footer { height: 91px; }\n    #chat_box { padding-bottom: 91px; }\n\n    #user { width: 100%; }\n    #message { width: 80%; }\n  }\n\n  @media(max-width: 400px) {\n    #footer { height: 135px; }\n    #chat_box { padding-bottom: 135px; }\n\n    #message { width: 100%; }\n    #send_btn {\n      position: relative;\n      margin-top: 3px;\n      width: 100%;\n    }\n  }\n</style>\n</head>\n\n<body>\n  <div id=\"wrapper\">\n    <div id=\"chat_box\" class=\"content\"></div>\n\n    <div id=\"footer\">\n      <div class=\"content\">\n        <input type=\"text\" id=\"user\" placeholder=\"Who are you?\" />\n        <input type=\"text\" id=\"message\" placeholder=\"What do you want to say?\" />\n        <input type=\"button\" id=\"send_btn\" value=\"Send\" onclick=\"sendMessage()\">\n        <input type=\"button\" id=\"clear_btn\" value=\"Clear\" onclick=\"clearMessages()\">\n      </div>\n    </div>\n  </div>\n</body>\n\n",
        "x": 304,
        "y": 128,
        "wires": [
            [
                "7f3f72c7d757f412"
            ]
        ]
    },
    {
        "id": "7f3f72c7d757f412",
        "type": "http response",
        "z": "6e675b81333ad533",
        "name": "",
        "x": 447,
        "y": 128,
        "wires": []
    },
    {
        "id": "bc740d23.438bf",
        "type": "websocket-listener",
        "path": "/ws/chat",
        "wholemsg": "false"
    }
]
```

### Forecast flow if you have GPS coordinates
```
[{"id":"0f60af10c76ec207","type":"tab","label":"ForecastFlow","disabled":false,"info":"","env":[]},{"id":"b2e8ef186edec15f","type":"aedes broker","z":"0f60af10c76ec207","name":"mqtt-broker","mqtt_port":1883,"mqtt_ws_bind":"port","mqtt_ws_port":"","mqtt_ws_path":"","cert":"","key":"","certname":"","keyname":"","dburl":"","usetls":false,"x":150,"y":300,"wires":[[],[]]},{"id":"30fab12f59a0d04d","type":"mqtt in","z":"0f60af10c76ec207","name":"","topic":"test/hello","qos":"2","datatype":"auto","broker":"f4579188cba3d931","nl":false,"rap":true,"rh":0,"inputs":0,"x":140,"y":720,"wires":[["d3ac56d1816a593a"]]},{"id":"a8ec2adf6caeb391","type":"mqtt out","z":"0f60af10c76ec207","name":"","topic":"test/hello","qos":"","retain":"","respTopic":"","contentType":"","userProps":"","correl":"","expiry":"","broker":"f4579188cba3d931","x":1740,"y":560,"wires":[]},{"id":"83770bd020198d6e","type":"inject","z":"0f60af10c76ec207","name":"","props":[{"p":"payload"},{"p":"topic","vt":"str"}],"repeat":"","crontab":"","once":false,"onceDelay":0.1,"topic":"","payload":"{\"latitude\":41.3122,\"longitude\":-73.8566}","payloadType":"json","x":150,"y":460,"wires":[["0ddcdbebeb61f1a1"]]},{"id":"a40884897e8308b9","type":"function","z":"0f60af10c76ec207","name":"","func":"msg.payload=\"Forecast is: \"+msg.payload\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":1580,"y":560,"wires":[["a8ec2adf6caeb391"]]},{"id":"d3ac56d1816a593a","type":"debug","z":"0f60af10c76ec207","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","statusVal":"","statusType":"auto","x":330,"y":720,"wires":[]},{"id":"76fc93ef88068239","type":"http request","z":"0f60af10c76ec207","name":"weather-request","method":"GET","ret":"obj","paytoqs":"ignore","url":"","tls":"","persist":false,"proxy":"","authType":"","senderr":false,"x":720,"y":560,"wires":[["de9b55534f1ab328"]]},{"id":"18ed439b14346dfd","type":"debug","z":"0f60af10c76ec207","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","statusVal":"","statusType":"auto","x":1590,"y":500,"wires":[]},{"id":"a316b596d6736118","type":"change","z":"0f60af10c76ec207","name":"shortForecast","rules":[{"t":"set","p":"payload","pt":"msg","to":"payload.properties.periods[0].shortForecast","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":1400,"y":560,"wires":[["18ed439b14346dfd","a40884897e8308b9","5dc60d49b84db6e3"]]},{"id":"de9b55534f1ab328","type":"change","z":"0f60af10c76ec207","name":"set-shortForecastURL","rules":[{"t":"set","p":"url","pt":"msg","to":"payload.properties.forecastHourly","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":940,"y":560,"wires":[["f32730c09d40debb","af8e87493683b96a"]]},{"id":"af8e87493683b96a","type":"http request","z":"0f60af10c76ec207","name":"shortForecast-request","method":"GET","ret":"obj","paytoqs":"ignore","url":"","tls":"","persist":false,"proxy":"","authType":"","senderr":false,"x":1180,"y":560,"wires":[["a316b596d6736118"]]},{"id":"f32730c09d40debb","type":"debug","z":"0f60af10c76ec207","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"url","targetType":"msg","statusVal":"","statusType":"auto","x":1140,"y":620,"wires":[]},{"id":"9f8e7493b1c548c1","type":"http in","z":"0f60af10c76ec207","name":"forecast","url":"/forecast","method":"get","upload":false,"swaggerDoc":"","x":140,"y":560,"wires":[["81e2d2f6cb4b5803","b3c62f3b35107f2a"]]},{"id":"5dc60d49b84db6e3","type":"template","z":"0f60af10c76ec207","name":"page","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"<html>\n<head>\n    <h1>Forecast</h1>\n</head>\n<body>\n    The forecast is: {{payload}} !\n</body>\n</html>","output":"str","x":1570,"y":620,"wires":[["6db639baea170317"]]},{"id":"6db639baea170317","type":"http response","z":"0f60af10c76ec207","name":"","statusCode":"","headers":{},"x":1730,"y":620,"wires":[]},{"id":"81e2d2f6cb4b5803","type":"debug","z":"0f60af10c76ec207","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"req.query","targetType":"msg","statusVal":"","statusType":"auto","x":340,"y":620,"wires":[]},{"id":"0ddcdbebeb61f1a1","type":"change","z":"0f60af10c76ec207","name":"set-weather-url","rules":[{"t":"set","p":"url","pt":"msg","to":"'https://api.weather.gov/points/' & msg.payload.latitude & ',' & msg.payload.longitude","tot":"jsonata"}],"action":"","property":"","from":"","to":"","reg":false,"x":520,"y":560,"wires":[["76fc93ef88068239","4b3860bf37ff7173"]]},{"id":"4b3860bf37ff7173","type":"debug","z":"0f60af10c76ec207","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"url","targetType":"msg","statusVal":"","statusType":"auto","x":700,"y":620,"wires":[]},{"id":"b3c62f3b35107f2a","type":"change","z":"0f60af10c76ec207","name":"set-payload","rules":[{"t":"set","p":"payload","pt":"msg","to":"req.query","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":330,"y":560,"wires":[["0ddcdbebeb61f1a1"]]},{"id":"6bcc358e8a5710c0","type":"comment","z":"0f60af10c76ec207","name":"For testing (no http response)","info":"Only for testing","x":200,"y":400,"wires":[]},{"id":"d769f8a1d5d9985c","type":"comment","z":"0f60af10c76ec207","name":"Two requests required for short forecast","info":"","x":230,"y":520,"wires":[]},{"id":"c4093a2f7c8375ee","type":"comment","z":"0f60af10c76ec207","name":"Receive a message in mqtt","info":"","x":190,"y":680,"wires":[]},{"id":"1dcaeb6447ef55ba","type":"comment","z":"0f60af10c76ec207","name":"Send message to mqtt","info":"","x":1920,"y":560,"wires":[]},{"id":"95d260ebeb05f159","type":"comment","z":"0f60af10c76ec207","name":"Return http response","info":"","x":1900,"y":620,"wires":[]},{"id":"bbbb2dc7c6a64437","type":"comment","z":"0f60af10c76ec207","name":"debug forecast","info":"","x":1780,"y":500,"wires":[]},{"id":"f4579188cba3d931","type":"mqtt-broker","name":"mqtt-broker","broker":"localhost","port":"1883","clientid":"","autoConnect":true,"usetls":false,"protocolVersion":"4","keepalive":"60","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","birthMsg":{},"closeTopic":"","closeQos":"0","closePayload":"","closeMsg":{},"willTopic":"","willQos":"0","willPayload":"","willMsg":{},"sessionExpiry":""}]
```

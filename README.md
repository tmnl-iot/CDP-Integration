# CDP-Integration
Node-red example flow for integration with TMNL - CDP for NB-IOT

In this example I used node-red to visualise the steps needed. For instructions on installation and manuals you can visit their website (https://nodered.org/). Node-red is a flow based programming environment based on node-js. Node-red is great for prototyping but not really well suited for production grade environments.Please note that securing node-red requires special attention by default your flow will be accessible to the public.


In the next part will explain the configuration of each of the nodes

Node 0)
You can use this test message when you have not yet an working CDP account or SIM card or hardware module.

Configure the JSON payload as

JSON
 Copy
{
    "reports": [
        {
            "serialNumber": "IMEI:358878080020537",
            "timestamp": 1524582877912,
            "subscriptionId": "fab4fabf-3e60-401a-bbea-cef815635d98",
            "resourcePath": "uplinkMsg/0/data",
            "value": "aabbccdd"
        }
    ],
    "registrations": [],
    "deregistrations": [],
    "updates": [],
    "expirations": [],
    "responses": []
}
{
    "reports": [
        {
            "serialNumber": "IMEI:358878080020537",
            "timestamp": 1524582877912,
            "subscriptionId": "fab4fabf-3e60-401a-bbea-cef815635d98",
            "resourcePath": "uplinkMsg/0/data",
            "value": "aabbccdd"
        }
    ],
    "registrations": [],
    "deregistrations": [],
    "updates": [],
    "expirations": [],
    "responses": []
}
Node 1)
This is HTTP receiver node is the application endpoint you register in the CDP. Using this example call.
Configure:
method : POST
url : /my_application

Application registration
 Copy
curl -X PUT \
  https://iot.netwerk.t-mobile.nl/m2m/applications/registration \
  -H 'Accept: application/json' \
  -H 'Authorization: Basic xxx' \
  -H 'Content-Type: application/json' \
  -d '{	"headers" : {	},
	"url" : "https://<this node red server>/my_application"
}
'
curl -X PUT \
  https://iot.netwerk.t-mobile.nl/m2m/applications/registration \
  -H 'Accept: application/json' \
  -H 'Authorization: Basic xxx' \
  -H 'Content-Type: application/json' \
  -d '{	"headers" : {	},
	"url" : "https://<this node red server>/my_application"
}
'
Node 2)
This change Node extracts the reports array from the message and discards the remainder.

Set msg.payload to msg.payload.reports

Node 3)
This split node outputs a separate message for every serialnumber. This node needs no further configuration.

Node 4)
This Javascript function node builds up the message according to the targets wishes.

In my case it contains;

JavaScript
 Copy
msg.timestamp = msg.payload.timestamp;
msg.serialNumber = msg.payload.serialNumber;   
msg.topic = msg.payload.serialNumber.trim();   
msg.resourcePath = msg.payload.resourcePath;
msg.payload.raw =  msg.payload.value;
    
return msg;
msg.timestamp = msg.payload.timestamp;
msg.serialNumber = msg.payload.serialNumber;   
msg.topic = msg.payload.serialNumber.trim();   
msg.resourcePath = msg.payload.resourcePath;
msg.payload.raw =  msg.payload.value;
    
return msg;
Node 5)
This optional Javascript function node converts your payload to a readable characters. But this is of course only if you send readable / printable data.

JavaScript
 Copy
// Convert hex to string 

var str = "";
for (var n=0; n<msg.payload.value.length; n+=2) {
    str += String.fromCharCode(parseInt(msg.payload.value.substr(n,2), 16));
}
msg.payload.value = str;
return msg;
// Convert hex to string 

var str = "";
for (var n=0; n<msg.payload.value.length; n+=2) {
    str += String.fromCharCode(parseInt(msg.payload.value.substr(n,2), 16));
}
msg.payload.value = str;
return msg;
And as a final step you can push the message forward to your preferred target. Possibly credentials or API keys should be added to the forwarded message.
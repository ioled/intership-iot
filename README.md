# Pasantia IoT iOLED

Documentación para práctica en el desarrollo de plataforma IoT

 
- MQTT

El protocolo MQTT se ha convertido en uno de los principales pilares del IoT por su sencillez y ligereza. MQTT son las siglas MQ Telemetry Transport. El funcionamiento del MQTT es un servicio de mensajería push con patrón publicador/suscriptor (pub-sub). Como vimos en la entrada anterior, en este tipo de infraestructuras los clientes se conectan con un servidor central denominado broker.

Para filtrar los mensajes que son enviados a cada cliente los mensajes se disponen en topics organizados jerárquicamente. Un cliente puede publicar un mensaje en un determinado topic. Otros clientes pueden suscribirse a este topic, y el broker le hará llegar los mensajes suscritos. Los clientes inician una conexión TCP/IP con el broker, el cual mantiene un registro de los clientes conectados. Esta conexión se mantiene abierta hasta que el cliente la finaliza. Por defecto, MQTT emplea el puerto 1883 y el 8883 cuando funciona sobre TLS.

  

MQTT dispone de un mecanismo de calidad del servicio o QoS, entendido como la forma de gestionar la robustez del envío de mensajes al cliente ante fallos (por ejemplo, de conectividad).

  

MQTT tiene tres niveles QoS posibles.

  

- QoS 0 unacknowledged (at most one): El mensaje se envía una única vez. En caso de fallo por lo que puede que alguno no se entregue.

- QoS 1 acknowledged (at least one): El mensaje se envía hasta que se garantiza la entrega. En caso de fallo, el suscriptor puede recibir algún mensaje duplicados.

- QoS 2 assured (exactly one). Se garantiza que cada mensaje se entrega al suscriptor, y únicamente una vez.

## Google cloud

En Google cloud estos es la documentación necesaria para publicar por MQTT.

  

https://cloud.google.com/iot/docs/samples/mqtt-samples

https://cloud.google.com/iot/docs/how-tos/mqtt-bridge#iot-core-mqtt-auth-run-nodejs

  
### Node.js
```javascript
/**
* @description Return the list of devices registered in Google Cloud IoT core.
* @param {Object} client - The google cloud client.
* @returns {Promise<Object[]>} Returns the list of devices.
*/

exports.getIotCoreDevices = async client => {
	const request = {parent: googleConf.iotCore.registryName};
	try {
        const {
            data: {devices},
        } = await client.projects.locations.registries.devices.list(request);
    return devices;
} catch (err) {
    console.log('Could not list devices');
    console.log(err);
    }
};

/**
* Send the configuration to a IoT core device.
* @param {Object} client The google cloud client.
* @param {String} deviceId The id or name of the devices registered.
* @param {string} config The configuration blob send to the device.
* @example
* // Configuration example
* { board: { led1: { duty: 1, state: false } } };

* @returns {Promise<number>} HTTP status code - 200, 429.

*/

exports.sendIotCoreDeviceConfig = async (client, deviceId, config) => {
    // Convert data to base64.
    const binaryData = Buffer.from(config).toString('base64');

    // Create request object.
    const request = {
        name: `${googleConf.iotCore.registryName}/devices/${deviceId}`,
        versionToUpdate: googleConf.iotCore.version,
        binaryData: binaryData,
    };
    try {
        // Send the request to iot core.
        const data = await client.projects.locations.registries.devices.modifyCloudToDeviceConfig(request);

        // Return 200 on success update.
        return data.status;
    } catch (err) {
        // return error 429.
        console.log('Response:', err);
        return err.response.status;
    }
};

  

/**
* Get the state of a iot core device.
* @param {Object} client The google cloud client.
* @param {String} deviceId The id or name of the devices registered.
* @returns {Promise<{deviceStates: Array<{binaryData: string}>}>} HTTP status code - 200, 429.
*/

exports.getIoTCoreDeviceState = async (client, deviceId) => {
    const request = {
        name: `${googleConf.iotCore.registryName}/devices/${deviceId}`,
    };
    try {
        const response = await client.projects.locations.registries.devices.states.list(request);
    return response.data;

    } catch (err) {
        console.log(err);
        return err.response.status;
    }
};
```

## Mongoose OS
An open source _Operating System_ for the Internet of Things. Supported microcontrollers: ESP32, ESP8266, STM32, TI CC3200, TI CC3220.

```javascript
load('api_mqtt.js');

// Topic to receive config.
let configTopic =  '/devices/'  + Cfg.get('device.id') +  '/config';
// Topic to send state data.
let stateTopic =  '/devices/'  + Cfg.get('device.id') +  '/state';
/**

* Subscribe to a MQTT topic and receive config data from IoT Core.
* @configTopic (str): mqtt topic to subscribe.
* @param  {string }  topic Mqtt topic.
* @param  {string}  msg config message from the cloud.
* @see  https://github.com/mongoose-os-libs/mqtt/blob/master/mjs_fs/api_mqtt.js
*/

let  connectMqtt  =  function() {
	print('Connecting to Mqtt topic: ', configTopic);
	MQTT.sub(configTopic, function(conn, topic, msg) {
		print('Topic:', topic, 'message:', msg);
	});
};

let res = MQTT.pub(stateTopic, JSON.stringify({temp: temp, hum: hum}), 1);
print('Published:', res ?  'yes'  :  'no');
```

```json
"board": {
	"led1": {
		"duty":0.85,
		"state":true
	},
	"led2": {
		"duty":0.85,
		"state":true
	},
	"timer": {
		"timerOn":"22:00",
		"timerOff":"10:00",
		"timerState":false
	}
}

```
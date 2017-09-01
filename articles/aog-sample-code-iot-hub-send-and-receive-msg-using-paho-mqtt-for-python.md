---
title: 使用 Paho MQTT For Python 发送接收 IOT Hub 消息示例代码
description: 使用 Paho MQTT For Python 发送接收 IOT Hub 消息示例代码
service: ''
resource: sample-code
author: chenrui1988
displayOrder: ''
selfHelpType: ''
supportTopicIds: ''
productPesIds: ''
resourceTags: 'Sample Code, IOT Hub, Paho MQTT For Python'
cloudEnvironments: MoonCake

ms.service: iot-hub
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 08/31/2017
wacn.date: 08/31/2017
---
# 使用 Paho MQTT For Python 发送接收 IOT Hub 消息示例代码

Azure IOT Hub 允许设备在端口 8883 上使用 MQTT V3.1.1 协议进行连接，但 Azure 要求所有的设备必须使用 TLS/SSL 保护设备的通信。

所以在使用 Paho MQTT 连接 IOT Hub 时，需要使用 TLS 连接，具体连接的方式参考以下步骤：

1.	需要安装 Paho MQTT for Python，使用 pip 安装 Paho :

    ```
    pip install  paho-mqtt
    ```

    更多安装的配置可以参考该组件官方说明：

    [Python Client - documentation](https://eclipse.org/paho/clients/python/docs/)

2.	下载 Azure IOT Hub 使用的证书，目前 Azure IOT Hub 使用的是 WoSign 证书，所以我们需要将 WoSign 根证书下载到本地，下载地址：[WoSign Root Certificates](https://www.wosign.com/english/root.htm)

    > [!NOTE]
    > Azure 将于 2017 年 9 月份之后，逐渐会将 WoSign 的证书替换为 DigiCert，届时，请将证书替换为 DigiCert 的根证书，详细请了解此博客 : [中国区 Azure IoT 中心服务的根证书变更通知](https://www.azure.cn/blog/2017/07/21/RootCertificateChangeNoticeforAzureIoTHubServiceinAzureinChina)

3.	编写 Python 代码，连接 Azure IOTHub，示例代码：

    ```Python
    import base64
    import hmac
    import urllib.parse

    import paho.mqtt.client as mqtt
    import time

    hubAddress = 'freeiottest.azure-devices.cn'
    deviceId = 'myDeviceId'
    deviceKey = '<your device key>'
    certPath = 'WS_CA1_NEW.crt'

    hubUser = hubAddress + '/' + deviceId
    endpoint = hubAddress + '/devices/' + deviceId
    hubTopicPublish = 'devices/' + deviceId + '/messages/events/'
    hubTopicSubscribe = 'devices/' + deviceId + '/messages/devicebound/#'


    def on_connect(client, userdata, flags, rc):
        print("Connected with result code "+str(rc))
        client.subscribe(hubTopicSubscribe)
        client.publish(hubTopicPublish, "testmessage")


    def on_message(client, userdata, msg):
        print("{0} - {1} ".format(msg.topic, str(msg.payload)))


    def generate_sas_token(uri, key, expiry=3600):
        ttl = int(time.time()) + expiry
        urlToSign = urllib.parse.quote(uri, safe='')
        sign_key = "%s\n%d" % (urlToSign, int(ttl))
        h = hmac.new(base64.b64decode(key), msg="{0}\n{1}".format(urlToSign, ttl).encode('utf-8'), digestmod='sha256')
        signature = urllib.parse.quote(base64.b64encode(h.digest()), safe='')
        return "SharedAccessSignature sr={0}&sig={1}&se={2}".format(urlToSign,urllib.parse.quote(base64.b64encode(h.digest()),safe=''), ttl)

    client = mqtt.Client(client_id=deviceId, protocol=mqtt.MQTTv311)
    client.on_connect = on_connect
    client.on_message = on_message
    client.tls_set(certPath)
    client.username_pw_set(username=hubUser,
                        password=generate_sas_token(endpoint, deviceKey))
    client.connect(hubAddress, port=8883, keepalive=60)

    client.loop_forever()
    ```

> [!NOTE]
> - 连接时，需要提供用户名和密码，密码使用 sas token 的形式。
> - 连接时，必须使用 tls 方式，即必须使用 tls_set 方法设置证书位置。

**参考文档** :

- [使用 MQTT 协议与 IoT 中心通信](https://docs.azure.cn/zh-cn/iot-hub/iot-hub-mqtt-support)

**示例代码** :

- [使用 Paho MQTT For Python 发送接收 IOT Hub 消息示例代码](https://github.com/wacn/AOG-CodeSample/blob/master/IotHub/Python/send-and-recevice-message-from-azure-iothub-%20using-paho-mqtt-for-python/send-and-recevice-message-from-azure-iothub-%20using-paho-mqtt-for-python.py)
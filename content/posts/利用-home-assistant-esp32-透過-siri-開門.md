---
title: 利用 Home Assistant + ESP32 透過 Siri 開門
date: 2022-10-17T18:10:33+08:00
draft: false
url: "/2022/10/17/利用-home-assistant-esp32-透過-siri-開門/"
categories:
  - 廢文
showToc: true
TocOpen: false
---

家裡的一個鐵捲門遙控器按鈕接觸不良，所以我買了一個新的，而舊的就可以直接拆下來玩了！ 手邊剛好有一套 4 路的繼電器模組，因此我就做了一點點改裝，達成透過手機遙控鐵門的開關！

## 硬體

硬體部份其實滿簡單的，開發版我採用了 ESP32S ，多了一個 S 最大的優點是，他的 GPIO 輸出為 5V，使用起來會比普通 ESP32 的 3.3V 方便很多。

把遙控器拆開後，透過電表量一下開關的接點，並直接把線焊出來接上繼電器即可，這邊沒有太大的技術難度。

![](https://i.imgur.com/pba24Ao.png)

## MQTT Server

ESP32 透過接收 MQTT 是最簡單的方法，我直接把 MQTT Server 架在我的 GCP 上，使用 Eclipse Mosquitto 的 Docker Compose

docker-compose.yml 由於我用不到 9001 Port，所以就將它著解掉

```yaml
version: "3"

services:
  mosquitto:
    image: eclipse-mosquitto
    volumes:
      - ./:/mosquitto/:rw
    ports:
      - 1883:1883
#      - 9001:9001
```

config/mosquitto.conf 則開啟了 password_file 的功能

```
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log

listener 1883
socket_domain ipv4
## Authentication ##
# allow_anonymous false
password_file /mosquitto/config/password.txt
```

接下來只需要透過 README.md 上面的方式設定帳密即可

```bash
docker-compose exec mosquitto mosquitto_passwd -b /mosquitto/config/password.txt user password
```

最後，跑熟悉得 `docker-compose up -d` 就可以開啟 MQTT Port 了

## Home Assistant

這邊我也採用了官方的 Docker File

Docker-compose 長這樣

```yaml
version: '3'
services:
  homeassistant:
    container_name: homeassistant
    image: "ghcr.io/home-assistant/home-assistant:stable"
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    privileged: true
    ports:
      - 127.0.0.1:8123:8123
```

之所以開在 127.0.0.1 是因為我想要透過 Reverse Proxy 方式綁 HTTPS，所以在 HA `config/configuration.yaml` 中添加以下內容， proxy 的 IP 改成自己 Docker 外的 IP

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 192.168.16.1
```

VHost 以及透過 certbot 申請 SSL 的過程就不贅述ㄌ，可以參考  
[https://towardsdatascience.com/how-to-host-multiple-website-with-apache-virtual-hosts-4423bd0aefbf](https://towardsdatascience.com/how-to-host-multiple-website-with-apache-virtual-hosts-4423bd0aefbf)

接下來設定 apache 的 config (`/etc/apache2/sites-available`) 檔案做 Rewrite

```apacheconf
ServerAdmin webmaster@localhost
    Servername  {ㄅ告訴你}

        ProxyPreserveHost On
        ProxyRequests Off
        ProxyPass /api/websocket ws://127.0.0.1:8123/api/websocket
        ProxyPassReverse /api/websocket wss://127.0.0.1:8123/api/websocket
        ProxyPass / http://127.0.0.1:8123/
        ProxyPassReverse / http://127.0.0.1:8123/

        #fix websockets for addons and apis
        RewriteEngine On
        RewriteCond %{HTTP:Upgrade} websocket [NC]
        RewriteRule ^/?(.*) "ws://127.0.0.1:8123/$1" [P,L]

        #Set security on certan areas(some redacted)
        
                Satisfy any
        
        
                Satisfy any
        

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

SSLCertificateFile /etc/letsencrypt/live/{ㄅ告訴你}/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/{ㄅ告訴你}/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
```

完成之後，再來設定 HA 接入 MQTT 的相關設定，在 HA `config/configuration.yaml` 中添加以下內容

```yaml
mqtt:
    broker: 自己的 Server
    username: 帳號
    password: 密碼
    button:
    - unique_id: home_door_open
      name: "Home Door Open Button"
      command_topic: "home/door"
      payload_press: "open"
    - unique_id: home_door_close
      name: "Home Door Close Button"
      command_topic: "home/door"
      payload_press: "close"
    - unique_id: home_door_pause
      name: "Home Door Pause Button"
      command_topic: "home/door"
      payload_press: "pause"
    - unique_id: home_door_disc
      name: "Home Door Disc Button"
      command_topic: "home/door"
      payload_press: "disc"
```

接下來透過瀏覽器登入 HA，照著 GUI 做一些常規的設定即可

![](https://i.imgur.com/S0UEqMB.png)

## ESP32 程式碼

```cpp
#include 
#include 

const char ssid[] = "喵";
const char pwd[] = "喵";

const char sub_id[] = "esp32s_1";
const char topic[] = "home/door";
const char mqtt_server[] = "喵";
const int mqtt_port = 1883;
const char mqtt_username[] = "喵";
const char mqtt_password[] = "喵";

const uint8_t up = 32;
const uint8_t stop = 33;
const uint8_t down = 25;
const uint8_t disc = 26;

WiFiClient espClient;
PubSubClient client(espClient);

unsigned long previousMillis = 0;
unsigned long interval = 30000;

void connect_wifi() {
  WiFi.mode(WIFI_STA);  //設置WiFi模式
  WiFi.begin(ssid, pwd);
  Serial.print("WiFi connecting");

  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }
  Serial.println("");
  Serial.print("IP Address:");
  Serial.println(WiFi.localIP());  //讀取IP位址
  Serial.print("WiFi RSSI:");
  Serial.println(WiFi.RSSI());  //讀取WiFi強度
}

void connect_mqtt() {
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(mqtt_callback);
  while (!client.connected()) {
    String client_id = "esp32-client-";
    client_id += String(WiFi.macAddress());
    Serial.printf("The client %s connects to my Mqtt broker\n", client_id.c_str());
    if (client.connect(client_id.c_str(), mqtt_username, mqtt_password)) {
      Serial.println("My Mqtt broker connected");
    } else {
      Serial.print("failed with state ");
      Serial.print(client.state());
      Serial.print(" \n");
      delay(2000);
    }
  }
  client.publish(topic, "Hi Meow meow I'm connected ^^");
  client.subscribe(topic);
  digitalWrite(LED_BUILTIN, 1);
}

void setup() {
  Serial.begin(115200);
  delay(5000);
  connect_wifi();
  connect_mqtt();

  pinMode(LED_BUILTIN, OUTPUT);
  pinMode(up, OUTPUT);
  pinMode(stop, OUTPUT);
  pinMode(down, OUTPUT);
  pinMode(disc, OUTPUT);

  digitalWrite(up, 1);
  digitalWrite(stop, 1);
  digitalWrite(down, 1);
  digitalWrite(disc, 1);
}

void touch(const uint8_t func) {
  digitalWrite(func, 0);
  delay(300);
  digitalWrite(func, 1);
  delay(300);
}

void mqtt_callback(char *topic, byte *payload, unsigned int length) {
  Serial.print("Message arrived in topic: ");
  Serial.println(topic);
  Serial.print("Message:");
  char payload_res[6] = { 0, 0, 0, 0, 0, 0 };
  for (int i = 0; i < length; i++) {
    payload_res[i] = (char)payload[i];
  }
  Serial.println(payload_res);
  if (!strcmp(payload_res, "open")) {
    touch(stop);
    touch(up);
  } else if (!strcmp(payload_res, "close")) {
    touch(stop);
    touch(down);
  } else if (!strcmp(payload_res, "pause")) {
    touch(stop);
  } else if (!strcmp(payload_res, "disc")) {
    touch(disc);
  }

  Serial.println();
  Serial.println("-----------------------");
}

void loop() {
  if (WiFi.status() != WL_CONNECTED) {
    digitalWrite(LED_BUILTIN, 0);
    Serial.print(millis());
    Serial.println("Reconnecting to WiFi...");
    WiFi.disconnect();
    connect_wifi();
    connect_mqtt();
  } else if (!client.connected()){
    digitalWrite(LED_BUILTIN, 0);
    connect_mqtt();
  }else {
    client.loop();
  }
}
```

程式碼內容滿簡單的，所以 BJ4，前半段是宣告， loop 中實作了 Wifi 斷線重連以及 MQTT 斷線重連。另外，我的繼電器模組是低電位觸發，所以跟直覺會有一點點反過來。

## Siri 控制 HA API

雖然 Apple 有 HomeKit 等裝置，但他的前提是需要有一台 APPLE TV 或是 iPad 在家，因此我採用的方法是建立一個 Long Lived Access Tokens

可以參考 ： [https://developers.home-assistant.io/docs/auth_api/](https://developers.home-assistant.io/docs/auth_api/)

![](https://i.imgur.com/3GLuDZa.png)

接下來其實就可以透過 curl / Postman 等方法直接透過 API 來控制了。

那麼，要如何接上 Siri 呢？我的方法是透過 iOS 的捷徑功能發送 HTTP Post，並帶上 Bearer 的 Access Token 即可完成。

![](https://i.imgur.com/yWFwkQV.png)

最後，再把捷徑匯入手機中，取個好記的名字，就可以對著手機喊「Hey Siri! 芝麻開門」來開門了！

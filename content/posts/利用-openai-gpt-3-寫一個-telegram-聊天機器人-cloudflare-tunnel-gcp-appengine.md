---
title: "利用 OpenAI GPT-3 寫一個 Telegram 聊天機器人 (Cloudflare Tunnel & GCP AppEngine)"
date: 2022-12-04T21:54:53+08:00
draft: false
url: "/2022/12/04/利用-openai-gpt-3-寫一個-telegram-聊天機器人-cloudflare-tunnel-gcp-appengine/"
categories:
  - 廢文
  - 小玩具
  - Cloud
showToc: true
TocOpen: false
---

最近 ChatGPT 很紅，想說可以試著把 OpenAI 的 API 給接上 Telegram 的群組來玩玩看，順便記錄一下 GCP 的 AppEngine Deploy 方法！

程式碼我放在：[https://github.com/stevenyu113228/OpenAI-GPT-3-Telegram-Chatbot](https://github.com/stevenyu113228/OpenAI-GPT-3-Telegram-Chatbot)

## 效果

![](https://i.imgur.com/cpJ0jy4.png)

![](https://i.imgur.com/tsFA5pd.png)

## OpenAI

先到 https://beta.openai.com/account/api-keys

申請一組 API Token

![](https://i.imgur.com/YrEdgSi.png)

在 [Playground](https://beta.openai.com/playground/p/default-chat) 上面隨意玩一下，複製他的 Example 來微調

![](https://i.imgur.com/ugWTPZ2.png)

```python
import os
import openai

openai.api_key = os.getenv("OPENAI_API_KEY")

start_sequence = "\nA:"
restart_sequence = "\n\nQ: "

response = openai.Completion.create(
  model="text-davinci-003",
  prompt="Q: ",
  temperature=0,
  max_tokens=100,
  top_p=1,
  frequency_penalty=0,
  presence_penalty=0,
  stop=["\n"]
)
```

## Telegram API

接下來到 Telegram 的 BotFather 來新增 Bot

![](https://i.imgur.com/tuNgxRF.png)

Botfather 會回應一組 API Token，把它抄下來

![](https://i.imgur.com/vwwppZd.png)

寫到我們的 Config (config.ini) 檔案裡面

```
[Telegram]
token=xxxxx:xxxxxxxx

[OPENAI]
key=xxxxxxxxxxxxxxxxxxxxx
```

接下來回到 Botfather 設定 `/setjoingroups` 為 `Enable`，以及 `/setprivacy` 為 `Disable`

![](https://i.imgur.com/aHphdVz.png)

## 完整程式碼

再來增加一點細節，串上 Telegram 的 API

```python
import openai
from flask import Flask, request
import telegram
from telegram.ext import Dispatcher, MessageHandler, Filters
import configparser

config = configparser.ConfigParser()
config.read("config.ini")

bot = telegram.Bot(token=(config['Telegram']['token']))
app = Flask(__name__)

@app.route('/hook', methods=['POST'])
def webhook_handler():
    if request.method == "POST":
        update = telegram.Update.de_json(request.get_json(force=True), bot)
        dispatcher.process_update(update)
    return 'ok'

openai.api_key = config['OPENAI']['key']

def chat_ai(input_str):
    response = openai.Completion.create(
    model="text-davinci-003",
    prompt=f"Human: {input_str} \n AI:",
    temperature=0.9,
    max_tokens=999,
    top_p=1,
    frequency_penalty=0,
    presence_penalty=0.6,
    stop=[" Human:", " AI:"]
    )

    print(response['choices'])
    res = response['choices'][0]['text']
    if res.startswith("？") or res.startswith("?"):
        res = res[1:]
    return res.strip()

def reply_handler(update ,bot):
    """Reply message."""
    try:
        text = update.message.text
        if text.startswith("機器人："):
          text = text[4:]
          print(text)
          res = chat_ai(text)
          print(res)
          update.message.reply_text(res)

    except Exception as e:
        print(e)

dispatcher = Dispatcher(bot, None)
dispatcher.add_handler(MessageHandler(Filters.text, reply_handler))

if __name__ == "__main__":
    # Running server
    app.run()
```

## CloudFlare Tunnel

接下來透過 cloudflared 開啟一個 tunnel 到 127.0.0.1:5000

```
cloudflared tunnel --url 127.0.0.1:5000
```

![](https://i.imgur.com/IibesH2.png)

並複製它吐回來的網址，如上圖是

```
https://baptist-freight-criteria-forums.trycloudflare.com
```

用 curl 或瀏覽器戳一次以下網址

```
https://api.telegram.org/bot{API_TOKEN}/setWebhook?url=https://{CLOUD_FLARE_TUNNEL}/hook
```

![](https://i.imgur.com/0Ifzr21.png)

接下來使用 `python3 main.py` 跑起來即可

![](https://i.imgur.com/snNCgZh.png)

## GCP

為了需要一個乾淨的環境取得 requirements.txt

我使用 docker

```
docker run -it --rm python:3.8 bash
pip3 install python-telegram-bot
pip3 install openai
pip3 install flask

pip3 freeze
```

這樣我們就能取得所有相關的 lib，建立一個 requirements.txt 把它存取來

```
APScheduler==3.6.3
backports.zoneinfo==0.2.1
cachetools==4.2.2
certifi==2022.9.24
charset-normalizer==2.1.1
click==8.1.3
et-xmlfile==1.1.0
Flask==2.2.2
idna==3.4
importlib-metadata==5.1.0
itsdangerous==2.1.2
Jinja2==3.1.2
MarkupSafe==2.1.1
numpy==1.23.5
openai==0.25.0
openpyxl==3.0.10
pandas==1.5.2
pandas-stubs==1.5.2.221124
python-dateutil==2.8.2
python-telegram-bot==13.14
pytz==2022.6
pytz-deprecation-shim==0.1.0.post0
requests==2.28.1
six==1.16.0
tornado==6.1
tqdm==4.64.1
types-pytz==2022.6.0.1
typing_extensions==4.4.0
tzdata==2022.7
tzlocal==4.2
urllib3==1.26.13
Werkzeug==2.2.2
zipp==3.11.0
```

再來建立一個 `app.yaml`

裡面輸入

```
service: openai-chatbot
runtime: python38
```

## GCP AppEngine (PaaS)

開啟 GCP Cloud Shell 的編輯器  
![](https://i.imgur.com/FUtoRzK.png)

把所有檔案都貼進來  
![](https://i.imgur.com/NiGipq7.png)

Cd 到資料夾後，輸入 `gcloud init`

![](https://i.imgur.com/8CzzFOc.png)

接下來輸入 `gcloud app deploy`

![](https://i.imgur.com/ePg2RsY.png)

等待它 Deploy 完畢，再跟前面一樣的方法設定 Webhook ，就能直接玩了ㄛ！

```
https://api.telegram.org/bot{API_TOKEN}/setWebhook?url=https://{APP Engine URL}/hook
```

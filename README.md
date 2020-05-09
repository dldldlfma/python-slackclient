# Python slackclient

Python slackclient는 Slack Web API와 Real Time Messaging(RTM) API의 interface를 제공하는 개발자 도구입니다. python 3.6 version 이상에서 동작합니다.

**Slack Python에 대한 설명문서는 오른쪽 주소에서 확인하실 수 있습니다. [https://slack.dev/python-slackclient/](https://slack.dev/python-slackclient/)**

[![pypi package][pypi-image]][pypi-url]
[![Build Status][travis-image]][travis-url]
[![Build Status][windows-build-status]][windows-build-url]
[![Python Version][python-version]][pypi-url]
[![codecov][codecov-image]][codecov-url]
[![contact][contact-image]][contact-url]

여러분이 당신의 팀을 위한 custom app을 만들거나 다른 third party service를 당신의 Slack workflows에 통합하려고 하면, Python 기반의 Slack Developer Kit은 python의 유연성을 활용해서 당신의 프로젝트가 가능한 빠르게 진행될수 있도록 도와줄겁니다!!

**Python slackclient**는 다음 것들과 상호작용 할 수 있게 해줍니다.

- Slack web api의 사용 [API Docs site][api-methods]
- 우리의 RTM과의 상호작용[RTM API][rtm-docs]

만약에 여러분이 [Event API][events-docs]를 사용하길 원하신다면, [Slack Events API adapter for Python][python-slack-events-api]를 확인해보세요.

Tokens과 Authentication(인증)에 관한 세부사항은 [Auth Guide](https://slack.dev/python-slackclient/auth.html) 에서 찾을 수 있습니다.

## Table of contents

* [준비사항](#준비사항)
* [설치](#installation)
* [tutorial 시작하기](#tutorial-시작하기)
* [Web Client 사용 기초](#Web-Client-사용-기초)
    * [Slack에 메세지 보내기](#Slack에-메세지-보내기)
    * [Slack에 file 업로드하기](#Slack에-file-업로드하기)
* [RTM Client 사용 기초](#basic-usage-of-the-rtm-client)
* [Async Usage](#async-usage)
    * [Slackclient as a script](#slackclient-as-a-script)
    * [Slackclient in a framework](#slackclient-in-a-framework)
* [고급 옵션](#advanced-options)
* [Migrating from v1.x](#migrating-from-v1)
* [지원](#support)

### 준비사항

---

이 library는 Python 3.6 이상의 버전이 필요합니다. 만약에 여러분이 Python 2를 사용중이라면, [SlackClient - v1.x][slackclientv1]을 사용해주세요. 만약 Python Version 확인법을 모르시면 아래 방법을 통해서 확인 가능합니다!

> **Note:** You may need to use `python3` before your commands to ensure you use the correct Python path. e.g. `python3 --version`

```bash
python --version

-- or --

python3 --version
```

### 설치

우리는 [PyPI][pypi]를 사용해서 Slack Developer Kit을 설치하는걸 추천드립니다.

```bash
$ pip3 install slackclient
```

### tutorial 시작하기

---

우리는 이 [tutorial[(/tutorial)을 10분보다 짧은 시간안에 기본적인 Slack app을 만들어 볼수 있도록 만들었습니다. 이걸 진행하는데 기본적인 프로그래밍 지식이 필요하고, 물론 Python에 대한 기본지식도 필요합니다. 이 tutorial은 Slack의 Web과 RTM API와 연결하는데 집중되어 있습니다. 이 tutorial을 통해서 어떻게 SDK를 활용해야 하는지 알아보세요!


**[Read the tutorial to get started!](/tutorial)**

### Web Client 사용 기초

---

Slack provide a Web API that gives you the ability to build applications that interact with Slack in a variety of ways. This Development Kit is a module based wrapper that makes interaction with that API easier. We have a basic example here with some of the more common uses but a full list of the available methods are available [here][api-methods]. More detailed examples can be found in our [Basic Usage](https://slack.dev/python-slackclient/basic_usage.html) guide

#### Slack에 메세지 보내기

One of the most common use-cases is sending a message to Slack. If you want to send a message as your app, or as a user, this method can do both. In our examples, we specify the channel name, however it is recommended to use the `channel_id` where possible.

```python
import os
from slack import WebClient
from slack.errors import SlackApiError

client = WebClient(token=os.environ['SLACK_API_TOKEN'])

try:
    response = client.chat_postMessage(
        channel='#random',
        text="Hello world!")
    assert response["message"]["text"] == "Hello world!"
except SlackApiError as e:
    # You will get a SlackApiError if "ok" is False
    assert e.response["ok"] is False
    assert e.response["error"]  # str like 'invalid_auth', 'channel_not_found'
    print(f"Got an error: {e.response['error']}")
```

Here we also ensure that the response back from Slack is a successful one and that the message is the one we sent by using the `assert` statement.

#### Slack에 file 업로드하기

We've changed the process for uploading files to Slack to be much easier and straight forward. You can now just include a path to the file directly in the API call and upload it that way. You can find the details on this api call [here][files.upload]

```python
import os
from slack import WebClient
from slack.errors import SlackApiError

client = WebClient(token=os.environ['SLACK_API_TOKEN'])

try:
    filepath="./tmp.txt"
    response = client.files_upload(
        channels='#random',
        file=filepath)
    assert response["file"]  # the uploaded file
except SlackApiError as e:
    # You will get a SlackApiError if "ok" is False
    assert e.response["ok"] is False
    assert e.response["error"]  # str like 'invalid_auth', 'channel_not_found'
    print(f"Got an error: {e.response['error']}")
```

### Basic Usage of the RTM Client

---

The [Real Time Messaging (RTM) API][rtm-docs] is a WebSocket-based API that allows you to receive events from Slack in real time and send messages as users.

If you prefer events to be pushed to you instead, we recommend using the HTTP-based [Events API][events-docs] instead. Most event types supported by the RTM API are also available in the Events API. You can check out our [Python Slack Events Adaptor][events-sdk] if you want to use this API instead.

An RTMClient allows apps to communicate with the Slack Platform's RTM API.

The event-driven architecture of this client allows you to simply
link callbacks to their corresponding events. When an event occurs
this client executes your callback while passing along any
information it receives. We also give you the ability to call our web client from inside your callbacks.

In our example below, we watch for a [message event][message-event] that contains "Hello" and if its received, we call the `say_hello()` function. We then issue a call to the web client to post back to the channel saying "Hi" to the user.

```python
import os
from slack import RTMClient
from slack.errors import SlackApiError

@RTMClient.run_on(event='message')
def say_hello(**payload):
    data = payload['data']
    web_client = payload['web_client']
    rtm_client = payload['rtm_client']
    if 'text' in data and 'Hello' in data.get('text', []):
        channel_id = data['channel']
        thread_ts = data['ts']
        user = data['user']

        try:
            response = web_client.chat_postMessage(
                channel=channel_id,
                text=f"Hi <@{user}>!",
                thread_ts=thread_ts
            )
        except SlackApiError as e:
            # You will get a SlackApiError if "ok" is False
            assert e.response["ok"] is False
            assert e.response["error"]  # str like 'invalid_auth', 'channel_not_found'
            print(f"Got an error: {e.response['error']}")

rtm_client = RTMClient(token=os.environ["SLACK_API_TOKEN"])
rtm_client.start()
```

### Async usage

slackclient v2 and higher uses aiohttp and asyncio to enable async functionality.

Normal usage of the library does not run it in async, hence a kwarg of run_async=True is needed.

When in async mode its important to remember to await or run/run_until_complete the call.

#### Slackclient as a script

```python 
import asyncio
import os
from slack import WebClient
from slack.errors import SlackApiError

client = WebClient(
    token=os.environ['SLACK_API_TOKEN'],
    run_async=True
)
future = client.chat_postMessage(
    channel='#random',
    text="Hello world!"
)

loop = asyncio.get_event_loop()
try:
    # run_until_complete returns the Future's result, or raise its exception.
    response = loop.run_until_complete(future)
    assert response["message"]["text"] == "Hello world!"
except SlackApiError as e:
    assert e.response["ok"] is False
    assert e.response["error"]  # str like 'invalid_auth', 'channel_not_found'
    print(f"Got an error: {e.response['error']}")
finally:
    loop.close()
```

#### Slackclient in a framework

If you are using a framework invoking the asyncio event loop like : sanic/jupyter notebook/etc.

```python
import os
from slack import WebClient
from slack.errors import SlackApiError

client = WebClient(
    token=os.environ['SLACK_API_TOKEN'],
    run_async=True # turn async mode on
)
# Define this as an async function
async def send_to_slack(channel, text):
    try:
        # Don't forget to have await as the client returns asyncio.Future
        response = await client.chat_postMessage(
            channel=channel,
            text=text
        )
        assert response["message"]["text"] == text
    except SlackApiError as e:
        assert e.response["ok"] is False
        assert e.response["error"]  # str like 'invalid_auth', 'channel_not_found'
        raise e

# https://sanicframework.org/
from sanic import Sanic
from sanic.response import json

app = Sanic()
# e.g., http://localhost:3000/?text=foo&text=bar
@app.route('/')
async def test(request):
    text = 'Hello World!'
    if 'text' in request.args:
        text = "\t".join(request.args['text'])
    try:
        await send_to_slack(channel="#random", text=text)
        return json({'message': 'Done!'})
    except SlackApiError as e:
        return json({'message': f"Failed due to {e.response['error']}"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3000)
```

### Advanced Options

The Python slackclient v2 now uses [AIOHttp][aiohttp] under the hood.

Looking for a performance boost? Installing the optional dependencies (aiodns) may help speed up DNS resolving by the client. We've included it as an extra called "optional":
```bash
$ pip3 install slackclient[optional]
```

Interested in SSL or Proxy support? Simply use their built-in [SSL](https://docs.aiohttp.org/en/stable/client_advanced.html#ssl-control-for-tcp-sockets) and [Proxy](https://docs.aiohttp.org/en/stable/client_advanced.html#proxy-support) arguments. You can pass these options directly into both the RTM and the Web client.

```python
import os
from slack import WebClient
from ssl import SSLContext

sslcert = SSLContext()
# pip3 install proxy.py
# proxy --port 9000 --log-level d
proxyinfo = "http://localhost:9000"

client = WebClient(
    token=os.environ['SLACK_API_TOKEN'],
    ssl=sslcert,
    proxy=proxyinfo
)
response = client.chat_postMessage(
    channel="#random",
    text="Hello World!")
print(response)
```

We will always follow the standard process in AIOHttp for those proxy and SSL settings so for more information, check out their documentation page linked [here][aiohttp].

### Migrating from v1

---

If you're migrating from v1.x of slackclient to v2.x, Please follow our migration guide to ensure your app continues working after updating.

**[Check out the Migration Guide here!](https://github.com/slackapi/python-slackclient/wiki/Migrating-to-2.x)**

### Support

---

If you get stuck, we’re here to help. The following are the best ways to get assistance working through your issue:

Use our [Github Issue Tracker][gh-issues] for reporting bugs or requesting features.
Visit the [Slack Community][slack-community] for getting help using Slack Developer Kit for Python or just generally bond with your fellow Slack developers.

<!-- Markdown links -->

[pypi-image]: https://badge.fury.io/py/slackclient.svg
[pypi-url]: https://pypi.python.org/pypi/slackclient
[windows-build-status]: https://ci.appveyor.com/api/projects/status/rif04t60ptslj32x/branch/master?svg=true
[windows-build-url]: https://ci.appveyor.com/project/slackapi/python-slackclient
[travis-image]: https://travis-ci.org/slackapi/python-slackclient.svg?branch=master
[travis-url]: https://travis-ci.org/slackapi/python-slackclient
[python-version]: https://img.shields.io/pypi/pyversions/slackclient.svg
[codecov-image]: https://codecov.io/gh/slackapi/python-slackclient/branch/master/graph/badge.svg
[codecov-url]: https://codecov.io/gh/slackapi/python-slackclient
[contact-image]: https://img.shields.io/badge/contact-support-green.svg
[contact-url]: https://slack.com/support
[api-docs]: https://api.slack.com
[slackclientv1]: https://github.com/slackapi/python-slackclient/tree/v1
[api-methods]: https://api.slack.com/methods
[rtm-docs]: https://api.slack.com/rtm
[events-docs]: https://api.slack.com/events-api
[events-sdk]: https://github.com/slackapi/python-slack-events-api
[message-event]: https://api.slack.com/events/message
[python-slack-events-api]: https://github.com/slackapi/python-slack-events-api
[pypi]: https://pypi.python.org/pypi
[pipenv]: https://pypi.org/project/pipenv/
[gh-issues]: https://github.com/slackapi/python-slackclient/issues
[slack-community]: http://slackcommunity.com/
[dev-roadmap]: https://github.com/slackapi/python-slackclient/wiki/Slack-Python-SDK-Roadmap
[migration-guide]: documentation_v2/Migration.md
[files.upload]: https://api.slack.com/methods/files.upload
[auth-guide]: documentation_v2/auth.md
[basic-usage]: documentation_v2/basic_usage.md
[aiohttp]: https://aiohttp.readthedocs.io/

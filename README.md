# django channels samplechat app

Quick start
-----------
1. Download the chat folder

2. move the chat folder to your django project

3. Add "chat" to your INSTALLED_APPS setting like this:
```
    INSTALLED_APPS = [
        ...
        'chat',
    ]
```

4. add chat app url to your django urls:
```
    url(r'^chat/', include('chat.urls')),
```
5. add the below routes to your django-channels route
```
    channel_routing = [
        route("websocket.connect", ws_add),
        route("websocket.receive", ws_message),
        route("websocket.disconnect", ws_disconnect),
    ]
```
6. add the below codes to your django-channels consumers
```
    @channel_session_user_from_http
    def ws_add(message):
        # Add them to the right group
        Group("chat").add(message.reply_channel)


    # Connected to websocket.receive
    @channel_session_user
    def ws_message(message):
        Group("chat").send({
            "text": json.dumps({
                "text": message['text'],
                "user": message.user.username,
            })
        })


    # Connected to websocket.disconnect
    @channel_session_user
    def ws_disconnect(message):
        Group("chat").send({
            "text": json.dumps({
                "text": "has left the room",
                "user": message.user.username,
            })
        })
        Group("chat").discard(message.reply_channel)
```
---
title: Implement channel-specific functionality | Microsoft Docs
description: Learn how to implement channel-specific functionality using the the Bot Builder SDK for .NET.
author: kbrandl
ms.author: v-kibran
manager: rstand
ms.topic: article
ms.prod: bot-framework
ms.date: 
ms.reviewer:
---

# Implement channel-specific functionality

Some channels provide features that cannot be implemented by using only [message text and attachments](bot-builder-dotnet-create-messages.md). To implement channel-specific functionality, you can pass native metadata to a channel in the `Activity` object's `ChannelData` property. For example, your bot can use the `ChannelData` property to instruct Telegram to send a sticker or to instruct Office365 to send an email.

This article describes how to use a message activity's `ChannelData` property to implement this channel-specific functionality:

| Channel | Functionality |
|----|----|
| Email | Send and receive an email that contains body, subject, and importance metadata |
| Slack | Send full fidelity Slack messages |
| Facebook | Send Facebook notifications natively |
| Telegram | Perform Telegram-specific actions, such as sharing a voice memo or a sticker |
| Kik | Send and receive native Kik messages | 

> [!NOTE]
> The value of an `Activity` object's `ChannelData` property is a JSON object. 
> Therefore, the examples in this article show the expected format of the 
> `channelData` JSON property in various scenarios. 
> To create a JSON object using .NET, use the `JObject` (.NET) class. 

## Create a custom Email message

To create an email message, set the `Activity` object's `ChannelData` property 
to a JSON object that contains these properties: 

| Property | Description |
|----|----|
| htmlBody | An HTML document that specifies the body of the email message. See the channel's documentation for information about supported HTML elements and attributes. |
| importance | The email's importance level. Valid values are **high**, **normal**, and **low**. The default value is **normal**. |
| subject | The email's subject. See the channel's documentation for information about field requirements. |

> [!NOTE]
> Messages that your bot receives from users via the Email channel may 
> contain a `ChannelData` property that is populated with a JSON object like the one described above.

This snippet shows an example of the `channelData` property for a custom email message.

```json
"channelData": {
    "htmlBody" : "<html><body style=\"font-family: Calibri; font-size: 11pt;\">This is the email body!</body></html>",
    "subject":"This is the email subject",
    "importance":"high"
}
```

## Create a full-fidelity Slack message

To create a full-fidelity Slack message, 
set the `Activity` object's `ChannelData` property to a JSON object that specifies 
<a href="https://api.slack.com/docs/messages" target="_blank">Slack messages</a>, 
<a href="https://api.slack.com/docs/message-attachments" target="_blank">Slack attachments</a>, and/or 
<a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack buttons</a>. 

> [!NOTE]
> To support buttons in Slack messages, you must enable **Interactive Messages** when you 
> [connect your bot](../portal-configure-channels.md) to the Slack channel.

This snippet shows an example of the `channelData` property for a custom Slack message.

```json
"channelData": {
   "text": "Now back in stock! :tada:",
   "attachments": [
        {
            "title": "The Further Adventures of Slackbot",
            "author_name": "Stanford S. Strickland",
            "author_icon": "https://api.slack.com/img/api/homepage_custom_integrations-2x.png",
            "image_url": "http://i.imgur.com/OJkaVOI.jpg?1"
        },
        {
            "fields": [
                {
                    "title": "Volume",
                    "value": "1",
                    "short": true
                },
                {
                    "title": "Issue",
                    "value": "3",
                    "short": true
                }
            ]
        },
        {
            "title": "Synopsis",
            "text": "After @episod pushed exciting changes to a devious new branch back in Issue 1, Slackbot notifies @don about an unexpected deploy..."
        },
        {
            "fallback": "Would you recommend it to customers?",
            "title": "Would you recommend it to customers?",
            "callback_id": "comic_1234_xyz",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "recommend",
                    "text": "Recommend",
                    "type": "button",
                    "value": "recommend"
                },
                {
                    "name": "no",
                    "text": "No",
                    "type": "button",
                    "value": "bad"
                }
            ]
        }
    ]
}
```

When a user clicks a button within a Slack message, your bot will receive a response message
in which the `ChannelData` property is populated with a `payload` JSON object. 
The `payload` object specifies contents of the original message, 
identifies the button that was clicked, and identifies the user who clicked the button. 

This snippet shows an example of the `channelData` property in the message that a bot receives 
when a user clicks a button in the Slack message.

```json
"channelData": {
    "payload": {
        "actions": [
            {
                "name": "recommend",
                "value": "yes"
            }
        ],
        . . .
        "original_message": "{…}",
        "response_url": "https://hooks.slack.com/actions/..."
    }
}
```

Your bot can reply to this message in the [normal manner](bot-builder-dotnet-connector.md#create-reply), 
or it can post its response directly to the endpoint that is specified by 
the `payload` object's `response_url` property.
For information about when and how to post a response to the `response_url`, see 
<a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack Buttons</a>. 

## Create a Facebook notification

To create a Facebook notification, 
set the `Activity` object's `ChannelData` property to a JSON object that specifies these properties: 

| Property | Description |
|----|----|
| notification_type | The type of notification (e.g., **REGULAR**, **SILENT_PUSH**, **NO_PUSH**).
| attachment | An attachment that specifies an image, video, or other multimedia type, or a templated attachment such as a receipt. |

> [!NOTE]
> For details about format and contents of the `notification_type` property and `attachment` property, see the 
> <a href="https://developers.facebook.com/docs/messenger-platform/send-api-reference#guidelines"target="_blank">Facebook API documentation</a>. 

This snippet shows an example of the `channelData` property for a Facebook receipt attachment.

```json
"channelData": {
    "notification_type": "NO_PUSH",
    "attachment": {
        "type": "template"
        "payload": {
            "template_type": "receipt",
            . . .
        }
    }
}
```

## Create a Telegram message

To create a message that implements Telegram-specific actions, 
such as sharing a voice memo or a sticker, 
set the `Activity` object's `ChannelData` property to a JSON object that specifies these properties: 

| Property | Description |
|----|----|
| method | The Telegram Bot API method to call. |
| parameters | The parameters of the specified method. |

These Telegram methods are supported: 

- answerInlineQuery
- editMessageCaption
- editMessageReplyMarkup
- editMessageText
- forwardMessage
- kickChatMember
- sendAudio
- sendChatAction
- sendContact
- sendDocument
- sendLocation
- sendMessage
- sendPhoto
- sendSticker
- sendVenue
- sendVideo
- sendVoice
- unbanChateMember

For details about these Telegram methods and their parameters, see the 
<a href="https://core.telegram.org/bots/api#available-methods" target="_blank">Telegram Bot API documentation</a>.

> [!NOTE]
> <ul><li>The `chat_id` parameter is common to all Telegram methods. If you do not specify `chat_id` as a parameter, the framework will provide the ID for you.</li>
<li>Instead of passing file contents inline, specify the file using a URL and media type as shown in the example below.</li>
<li>Within each message that your bot receives from the Telegram channel, the `ChannelData` property will include the message that your bot sent previously.</li></ul>

This snippet shows an example of a `channelData` property that specifies a single Telegram method.

```json
"channelData": {
    "method": "sendSticker",
    "parameters": {
        "sticker": {
            "url": "https://domain.com/path/gif",
            "mediaType": "image/gif",
        }
    }
}
```

This snippet shows an example of a `channelData` property that specifies an array of Telegram methods.

```json
"channelData": [
    {
        "method": "sendSticker",
        "parameters": {
            "sticker": {
                "url": "https://domain.com/path/gif",
                "mediaType": "image/gif",
            }
        }
    },
    {
        "method": "sendMessage",
        "parameters": {
            "text": "<b>This message is HTML formatted.</b>",
            "parse_mode": "HTML"
        }
    }
]
```

## Create a native Kik message

To create a native Kik message, 
set the `Activity` object's `ChannelData` property to a JSON object that specifies this property: 

| Property | Description |
|----|----|
| messages | An array of Kik messages. For details about Kik message format, see <a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">Kik Message Formats</a>. |

This snippet shows an example of the `channelData` property for a native Kik message.

```json
"channelData": {
    "messages": [
        {
            "chatId": "c6dd8165…",
            "type": "link",
            "to": "kikhandle",
            "title": "My Webpage",
            "text": "Some text to display",
            "url": "http://botframework.com",
            "picUrl": "http://lorempixel.com/400/200/",
            "attribution": {
                "name": "My App",
                "iconUrl": "http://lorempixel.com/50/50/"
            },
            "noForward": true,
            "kikJsData": {
                    "key": "value"
                }
        }
    ]
}
```

## Additional resources

- [Activities overview](bot-builder-dotnet-activities.md)
- [Create messages](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.activity?view=botbuilder-3.8" target="_blank">Activity class</a>

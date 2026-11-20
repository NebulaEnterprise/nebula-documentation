# Nebula Bot API

**Version:** `1.0.0`  
**Platform:** Nebula  
**SDK:** JavaScript (Node.js)  
**Minimum Node.js:** 18+

The official documentation for developing, managing, and integrating bots with the Nebula platform.

---

## Table of Contents

- [Overview](#overview)
- [Getting Started](#getting-started)
- [Authentication](#authentication)
- [Quick Start](#quick-start)
- [Bot Objects](#bot-objects)
- [JavaScript SDK](#javascript-sdk)
- [Bot HTTP API](#bot-http-api)
  - [Version](#get-version)
  - [Bot Profile](#get-me)
  - [Update Profile](#patch-me)
  - [Update Avatar](#put-meavatar)
  - [Commands](#get-commands)
  - [Events](#get-events)
  - [Servers](#get-servers)
  - [Server Channels](#get-server-channels)
  - [Channel Information](#get-channel)
  - [Message History](#get-messages)
  - [Send Messages](#post-messages)
- [Account API](#account-api)
- [Events Reference](#events-reference)
- [Examples](#examples)
- [Errors and Limits](#errors-and-limits)
- [Security](#security)

---

## Overview

Nebula bots are independent accounts that interact with the platform through an HTTPS API.

A bot is not hosted by Nebula. Instead, developers run their own bot application, which communicates with Nebula using its bot token.

The bot application maintains a long-polling connection to receive events and sends requests back to the API when it needs to perform actions.

**How it works:**

1. Create a bot through Nebula Developer Settings.
2. Generate and securely store its authentication token.
3. Add the bot to a server.
4. Run your bot application using Node.js or another HTTP-compatible environment.
5. Receive events from Nebula.
6. Respond to messages and interact with servers through the API.

Closing the bot application stops the bot from receiving events and performing actions.

### API Endpoints

| API | Base URL |
|---|---|
| Bot HTTP API | `https://scowling-anthill-civil.ngrok-free.dev/api/bot/v1` |
| Account API | `https://scowling-anthill-civil.ngrok-free.dev/api` |

The Bot HTTP API uses bot-token authentication.

The Account API uses your signed-in Nebula session and is primarily intended for managing bots through Settings and Server Settings.

### Bot Limitations

| Limit | Maximum |
|---|---:|
| Bots per account | 2 |
| Bots per server | 25 |
| Commands per bot | 100 |
| Messages per bot | 30/minute |
| Message length | 2,000 characters |
| Embeds per message | 4 |
| Avatar file size | 8 MB |
| Event results per poll | 100 |

A bot can only access servers it has been added to and channels it has permission to view.

Additional restrictions:

- Bots cannot sign in through the Nebula website.
- Bots do not receive their own messages as events.
- Webhook messages are not delivered as bot message events.
- Sealed messages are not delivered as bot message events.
- Voice channels cannot receive messages.
- Each bot has one active authentication token.

---

## Getting Started

### Requirements

Before creating a bot, make sure you have:

- A Nebula account.
- Permission to manage the server where the bot will be added.
- Node.js 18 or newer.
- A code editor such as Visual Studio Code.
- A bot token generated through Nebula Developer Settings.

### Creating a Bot

1. Open Nebula.
2. Navigate to **Settings → Developer**.
3. Enter a name for your bot.
4. Press **Create**.
5. Copy the generated bot token.
6. Add the bot to a server you manage.

**Important:** The bot token is displayed only once when generated. Store it somewhere secure. You cannot retrieve the original token later.

If you lose your token, generate a new one through Developer Settings.

Generating a new token immediately invalidates the previous token.

---

## Authentication

All Bot HTTP API endpoints require authentication except `GET /version`.

Include the following HTTP header in authenticated requests:

```http
Authorization: Bot nb_PASTE_YOUR_TOKEN
```

The authentication format is:

```text
Authorization: Bot YOUR_BOT_TOKEN
```

The token must begin with `nb_`.

### Important

Do not confuse bot authentication with account authentication.

| Authentication | Header | Used for |
|---|---|---|
| Bot token | `Authorization: Bot TOKEN` | Bot HTTP API |
| Account session | `Authorization: Bearer SESSION` | Account API |

The `Bearer` authentication method is reserved for signed-in Nebula accounts.

Using a bot token with `Bearer` authentication will not work.

### Content Type

JSON requests must include:

```http
Content-Type: application/json
```

The avatar upload endpoint is an exception. It accepts raw image bytes instead of JSON.

### API Version Header

Every bot API response includes:

```http
X-Nebula-API-Version: 1.0.0
```

---

## Quick Start

This example creates a basic bot that responds to `/ping` and `!ping`.

### 1. Project Structure

Create a folder containing:

```text
my-nebula-bot/
│
├── nebula.js
├── starter.js
└── package.json
```

Download the following files from the Nebula bot SDK:

- `nebula.js`
- `starter.js`
- `package.json`

### 2. package.json

```json
{
  "name": "my-nebula-bot",
  "version": "1.0.0",
  "type": "module",
  "description": "A Nebula bot",
  "scripts": {
    "start": "node starter.js"
  }
}
```

The `"type": "module"` setting enables ES module imports.

### 3. starter.js

```javascript
import { Bot } from './nebula.js';

const bot = new Bot('nb_PASTE_YOUR_TOKEN');

bot.command('ping', 'Is the bot alive?', (msg) => {
  msg.reply('pong');
});

bot.on('message', (msg) => {
  if (msg.mentioned) {
    msg.reply('hey');
  }
});

bot.on('ready', (me) => {
  console.log('online as', me.username);
});

bot.on('error', (err) => {
  console.error(err.status, err.code, err.message);
});

bot.start();
```

### 4. Start the Bot

Open a terminal inside your project folder:

```bash
node starter.js
```

Alternatively:

```bash
npm start
```

If everything works, the terminal will display something similar to:

```text
online as welcome
```

### 5. Test the Bot

Open a server where your bot has been added.

Send:

```text
/ping
```

The bot should respond:

```text
pong
```

You can also use:

```text
!ping
```

Both command formats are supported.

**Troubleshooting:** If nothing happens, open Settings → Developer, select your bot, and press **Send test event**. A running bot should receive the test event.

Keep the terminal running while you want the bot online.

---

# Bot Objects

This section describes the objects returned by the Bot HTTP API and JavaScript SDK.

## Bot

Returned by:

- `GET /me`
- `PATCH /me`
- `PUT /me/avatar`

| Field | Type | Description |
|---|---|---|
| `id` | string | Bot account ID. |
| `username` | string | Unique login name generated from the display name. |
| `display_name` | string | Display name shown in chat. |
| `displayName` | string | Alternative field for display name. |
| `bio` | string | Bot biography, up to 300 characters. |
| `avatar_url` | string | Bot avatar URL, or an empty string. |
| `bot` | boolean | Always `true`. |
| `owner_id` | string | Account ID of the bot owner. |
| `name` | string | Same as the stored display name. |

Bots cannot use their generated username to sign in to Nebula.

## Channel

| Field | Type | Description |
|---|---|---|
| `id` | string | Channel ID. |
| `name` | string | Channel name without `#`. |
| `type` | string | `text` or `voice`. |
| `server_id` | string | ID of the server containing the channel. |

Voice channels do not support sending messages.

## History Message

Returned by `GET /channels/:id/messages`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Message ID. |
| `channel_id` | string | Channel containing the message. |
| `author_id` | string | Author account ID. Webhook IDs start with `wh:`. |
| `author` | string | Author display name. |
| `content` | string | Message content. Empty when the message is sealed. |
| `created_at` | string | ISO 8601 timestamp. |
| `bot` | boolean | Whether the author is a bot or webhook. |

## Sent Message

Returned inside the response from `POST /channels/:id/messages`.

| Field | Type | Description |
|---|---|---|
| `id` | string | Newly created message ID. |
| `channel_id` | string | Destination channel ID. |
| `content` | string | Stored message content, limited to 2,000 characters. |
| `embeds` | array | Stored message embeds. |

## Embed

Embeds are rich message cards containing a colored sidebar, title, description, and optional URL.

A message supports up to four embeds.

| Field | Type | Limit | Description |
|---|---|---:|---|
| `title` | string | 256 characters | Embed title. |
| `description` | string | 2,000 characters | Embed description. |
| `url` | string | 500 characters | Optional HTTP or HTTPS link. |
| `color` | string or number | — | Embed sidebar color. |

A valid embed must contain at least one of:

- `title`
- `description`
- `url`

Colors can be provided as hexadecimal strings or numeric values.

Example:

```json
{
  "title": "Welcome",
  "description": "Welcome to Nebula!",
  "url": "https://nebulaapp.space",
  "color": "#c084fc"
}
```

The API stores the color as a number.

A color value of `0` means no custom sidebar color.

## Command

| Field | Type | Limit | Description |
|---|---|---:|---|
| `name` | string | 1–32 characters | Command name. |
| `description` | string | 100 characters | Command description. |

Command names support:

- Lowercase letters: `a-z`
- Numbers: `0-9`
- Hyphens: `-`
- Underscores: `_`

Command names are stored in lowercase.

## Event Envelope

Each event returned by `GET /events` uses the following structure:

| Field | Type | Description |
|---|---|---|
| `id` | number | Sequential event ID. |
| `type` | string | Event type. |
| `data` | object | Event-specific payload. |
| `created_at` | string | Event creation timestamp. |

The `id` is used as the `after` parameter when requesting subsequent events.

The JavaScript SDK copies the event data onto the listener object and adds `type`, `reply`, and `say` when the event contains a channel.

---

# JavaScript SDK

The Nebula JavaScript SDK is distributed as a single file:

```text
nebula.js
```

It does not require additional npm dependencies.

The SDK handles:

- Authentication.
- Bot profile loading.
- Command registration.
- Event polling.
- Message sending.
- Event listeners.
- Automatic polling retries.
- Error handling.

## Initializing a Bot

```javascript
import { Bot } from './nebula.js';

const bot = new Bot('nb_PASTE_YOUR_TOKEN');
```

### Constructor

```javascript
new Bot(token, opts)
```

| Parameter | Type | Description |
|---|---|---|
| `token` | string | Bot authentication token. |
| `opts.api` | string | Custom API base URL. |
| `opts.log` | boolean | Set to `false` to hide the online message. |

If the token is empty, the SDK reads:

```text
NEBULA_BOT_TOKEN
```

The default API URL can also be overridden using:

```text
NEBULA_API_URL
```

or:

```text
NEBULA_API
```

## SDK Methods

| Method | Description |
|---|---|
| `bot.command(name, description, fn)` | Registers a command. |
| `bot.on(event, fn)` | Registers an event listener. |
| `bot.send(channelId, text, opts)` | Sends a message. |
| `bot.setProfile(profile)` | Updates the bot profile. |
| `bot.setAvatar(file)` | Updates the bot avatar. |
| `bot.servers()` | Returns the servers the bot belongs to. |
| `bot.api(method, path, body)` | Makes a direct API request. |
| `bot.start()` | Loads the bot and starts event polling. |
| `bot.stop()` | Stops event polling after the current request returns. |

### bot.command()

```javascript
bot.command(name, description, fn)
```

Registers a command that can be triggered using `/` or `!`.

Example:

```javascript
bot.command('hello', 'Say hello', (msg) => {
  msg.reply('Hello!');
});
```

Commands can be called using:

```text
/hello
!hello
```

Registering another command with the same name replaces the previous registration.

Returns the bot instance.

### bot.on()

```javascript
bot.on(event, fn)
```

Registers an event listener.

Supported events:

```text
message
member_join
server_add
server_remove
test
ready
error
```

Example:

```javascript
bot.on('member_join', (event) => {
  console.log(event.username, 'joined the server');
});
```

Listeners execute in the order they were registered.

A thrown listener error is forwarded to the local `error` event.

Returns the bot instance.

### bot.send()

```javascript
bot.send(channelId, text, opts)
```

Sends a message to a channel.

Example:

```javascript
await bot.send('CHANNEL_ID', 'Hello from Nebula!');
```

Options:

| Option | Description |
|---|---|
| `replyTo` | Message ID to reply to. |
| `embeds` | Array of embeds. |

Example with an embed:

```javascript
await bot.send('CHANNEL_ID', 'Scoreboard', {
  embeds: [
    {
      title: 'Match result',
      description: 'Blue won 3-1.',
      color: '#7c5cff'
    }
  ]
});
```

### bot.setProfile()

```javascript
bot.setProfile({
  displayName: 'Welcome',
  bio: 'Greets new members.'
});
```

Updates the bot's display name or biography.

Fields that are omitted remain unchanged.

Returns the updated bot object and stores it in:

```javascript
bot.me
```

### bot.setAvatar()

```javascript
await bot.setAvatar('./avatar.png');
```

Accepts:

- A file path string.
- Raw image bytes.

Supported formats:

- PNG
- JPG
- GIF
- WebP

Maximum file size: 8 MB.

### bot.servers()

```javascript
const servers = await bot.servers();

console.log(servers);
```

Returns the server array directly rather than the complete API response object.

### bot.api()

```javascript
bot.api(method, path, body)
```

Allows direct access to Bot HTTP API routes.

Example:

```javascript
const data = await bot.api(
  'GET',
  '/servers'
);

console.log(data.servers);
```

Supported body formats:

- JSON objects.
- Buffer.
- Uint8Array.

Buffer and Uint8Array values are sent as raw bytes.

Failed requests throw an Error containing:

```javascript
err.status
err.code
err.message
```

### bot.start()

```javascript
bot.start();
```

Starts the bot.

The SDK:

1. Validates the bot token.
2. Loads the bot profile through `/me`.
3. Registers the commands defined in your application.
4. Emits the local `ready` event.
5. Starts polling `/events`.

The event poll uses:

```text
after
wait=25
```

### bot.stop()

```javascript
bot.stop();
```

Stops the polling loop after the current poll request finishes.

### Message Methods

Message events receive additional helper methods.

| Method / Property | Description |
|---|---|
| `msg.reply(text, opts)` | Replies to the original message. |
| `msg.say(text, opts)` | Sends a message without quoting the original. |
| `msg.args` | Everything after the command name. |
| `msg.argv` | Arguments split by spaces. |

Example:

```javascript
bot.command('echo', 'Repeat your message', (msg) => {
  msg.reply(msg.args || 'Nothing to repeat.');
});
```

### Local Events

The following events are generated by the SDK rather than returned by the server:

**ready**

Receives the bot object after successful initialization.

```javascript
bot.on('ready', (me) => {
  console.log('Online as', me.username);
});
```

**error**

Receives an Error containing:

```javascript
err.message
err.status
err.code
```

```javascript
bot.on('error', (err) => {
  console.error(err);
});
```

### Polling and Retry Behavior

If a poll request fails, the SDK automatically retries.

| Response | Retry delay |
|---|---:|
| HTTP 429 | 5 seconds |
| Other failures | 2 seconds |

---

# Bot HTTP API

**Base URL**

```text
https://scowling-anthill-civil.ngrok-free.dev/api/bot/v1
```

All paths below are relative to the base URL.

Replace `:id` with the appropriate resource ID.

IDs containing characters other than letters, numbers, and hyphens should be URL-encoded.

## GET /version

Returns the current API and SDK versions.

**Authentication:** Not required.

Request:

```http
GET /version
```

Response:

```json
{
  "api": "1.0.0",
  "sdk": {
    "js": "1.0.0"
  }
}
```

The response also includes:

```http
X-Nebula-API-Version: 1.0.0
```

---

## GET /me

Returns the authenticated bot profile.

**Authentication:** Required.

Request:

```http
GET /me
Authorization: Bot nb_PASTE_YOUR_TOKEN
```

Response:

```json
{
  "id": "bot-id",
  "username": "welcome",
  "display_name": "Welcome",
  "displayName": "Welcome",
  "bio": "",
  "avatar_url": "",
  "bot": true,
  "owner_id": "owner-id",
  "name": "Welcome"
}
```

A successful authenticated request also marks the bot online for 50 seconds.

**Errors**

| Status | Code | Description |
|---|---|---|
| 401 | `unauthorized` | Missing or invalid bot token. |

---

## PATCH /me

Updates the bot's display name, biography, or both.

**Authentication:** Required.

Request:

```http
PATCH /me
Authorization: Bot nb_PASTE_YOUR_TOKEN
Content-Type: application/json
```

Body:

```json
{
  "display_name": "Welcome",
  "bio": "Greets new members."
}
```

Parameters:

| Field | Type | Limit |
|---|---|---|
| `display_name` | string | 1–32 characters |
| `displayName` | string | Alternative display name field |
| `bio` | string | 300 characters |

Rules:

- Display names are trimmed.
- Empty display names are rejected.
- Biography text is trimmed and limited to 300 characters.
- Send an empty string to clear the biography.
- Omitted fields remain unchanged.

Response:

Returns the updated bot object.

**Errors**

| Status | Code | Description |
|---|---|---|
| 400 | `invalid_request` | Invalid or missing display name. |

---

## PUT /me/avatar

Replaces the bot's avatar.

**Authentication:** Required.

Request:

```http
PUT /me/avatar
Authorization: Bot nb_PASTE_YOUR_TOKEN
```

The request body must contain the raw image bytes.

Supported formats:

- PNG
- JPG
- GIF
- WebP

Maximum file size: 8 MB.

Response:

```json
{
  "ok": true,
  "id": "bot-id",
  "username": "welcome"
}
```

The response includes the updated bot object.

**Errors**

| Status | Code | Description |
|---|---|---|
| 400 | `invalid_request` | Unsupported image format. |
| 413 | `too_large` | Image exceeds 8 MB. |
| 502 | `unavailable` | Image could not be stored. |

---

## GET /commands

Returns the commands registered to the bot.

**Authentication:** Required.

Request:

```http
GET /commands
Authorization: Bot nb_PASTE_YOUR_TOKEN
```

Response:

```json
{
  "commands": [
    {
      "name": "ping",
      "description": "Is the bot alive?"
    }
  ]
}
```

---

## PUT /commands

Replaces the entire command list.

**Authentication:** Required.

Request:

```http
PUT /commands
Authorization: Bot nb_PASTE_YOUR_TOKEN
Content-Type: application/json
```

Body:

```json
{
  "commands": [
    {
      "name": "ping",
      "description": "Is the bot alive?"
    },
    {
      "name": "setwelcome",
      "description": "Welcome new people in this channel"
    }
  ]
}
```

Rules:

- Maximum 100 commands.
- Command names must be 1–32 characters.
- Names support `a-z`, `0-9`, `-`, and `_`.
- Names are converted to lowercase.
- Duplicate names are rejected.

Commands are triggered by messages beginning with `/name` or `!name`.

There is currently no slash-command autocomplete menu.

Response:

```json
{
  "commands": [
    {
      "name": "ping",
      "description": "Is the bot alive?"
    }
  ]
}
```

**Errors**

| Status | Code | Description |
|---|---|---|
| 400 | `invalid_request` | More than 100 commands, invalid name, or duplicate command. |

---

## GET /events

Retrieves events for the authenticated bot.

**Authentication:** Required.

Request:

```http
GET /events?after=0&wait=25
Authorization: Bot nb_PASTE_YOUR_TOKEN
```

Parameters:

| Parameter | Type | Default | Description |
|---|---|---:|---|
| `after` | number | 0 | Returns events with IDs greater than this value. |
| `wait` | number | 0 | Long-polling duration in seconds. |

The maximum wait time is 25 seconds.

Each response contains at most 100 events.

Response:

```json
{
  "events": [
    {
      "id": 12,
      "type": "message",
      "data": {
        "object": "message",
        "content": "/ping"
      },
      "created_at": "2026-11-20T05:00:00.000Z"
    }
  ],
  "last": 12
}
```

Use the returned `last` value as the next `after` parameter.

The JavaScript SDK handles this automatically.

---

## GET /servers

Returns all servers the bot has been added to.

**Authentication:** Required.

Request:

```http
GET /servers
Authorization: Bot nb_PASTE_YOUR_TOKEN
```

Response:

```json
{
  "servers": [
    {
      "id": "server-id",
      "name": "Nebula Official",
      "member_count": 5
    }
  ]
}
```

---

## GET /servers/:id/channels

Returns channels the bot is permitted to see in a specific server.

**Authentication:** Required.

Request:

```http
GET /servers/SERVER_ID/channels?type=text
Authorization: Bot nb_PASTE_YOUR_TOKEN
```

Parameters:

| Parameter | Type | Description |
|---|---|---|
| `id` | string | Server ID. |
| `type` | string | Optional channel filter: `text` or `voice`. |

Response:

```json
{
  "channels": [
    {
      "id": "channel-id",
      "name": "welcome",
      "type": "text",
      "server_id": "server-id"
    }
  ]
}
```

**Errors**

| Status | Code | Description |
|---|---|---|
| 404 | `not_found` | Bot is not in the specified server. |

---

## GET /channels/:id

Returns information about a specific channel.

**Authentication:** Required.

Request:

```http
GET /channels/CHANNEL_ID
Authorization: Bot nb_PASTE_YOUR_TOKEN
```

Response:

```json
{
  "id": "channel-id",
  "name": "welcome",
  "type": "text",
  "server_id": "server-id"
}
```

**Errors**

| Status | Code | Description |
|---|---|---|
| 404 | `not_found` | Channel does not exist or is not visible to the bot. |

---

## GET /channels/:id/messages

Returns recent messages from a channel.

**Authentication:** Required.

Request:

```http
GET /channels/CHANNEL_ID/messages?limit=50
Authorization: Bot nb_PASTE_YOUR_TOKEN
```

Parameters:

| Parameter | Type | Default | Description |
|---|---|---:|---|
| `limit` | number | 50 | Number of messages to return. |

Limits:

- Minimum: 1.
- Maximum: 100.

Messages are returned oldest first within the selected page.

Sealed messages have an empty `content` field.

This endpoint does not return embeds.

Response:

```json
{
  "messages": [
    {
      "id": "message-id",
      "channel_id": "channel-id",
      "author_id": "user-id",
      "author": "Nova",
      "content": "hello",
      "created_at": "2026-11-20T05:00:00.000Z",
      "bot": false
    }
  ]
}
```

**Errors**

| Status | Code | Description |
|---|---|---|
| 404 | `not_found` | Channel does not exist or is not visible. |

---

## POST /channels/:id/messages

Sends a message as the authenticated bot.

**Authentication:** Required.

Request:

```http
POST /channels/CHANNEL_ID/messages
Authorization: Bot nb_PASTE_YOUR_TOKEN
Content-Type: application/json
```

Body:

```json
{
  "content": "Hello from Nebula!"
}
```

### Parameters

| Field | Type | Limit | Description |
|---|---|---:|---|
| `content` | string | 2,000 characters | Message content. |
| `reply_to` | string | — | Optional message ID to reply to. |
| `embeds` | array | 4 embeds | Optional embed cards. |

You can send:

- Text only.
- Embeds only.
- Text and embeds together.

An empty message with no valid embeds is rejected.

### Example: Text Message

```json
{
  "content": "Hello from Nebula!"
}
```

### Example: Reply

```json
{
  "content": "That is correct!",
  "reply_to": "message-id"
}
```

The `reply_to` ID must belong to the same channel. Unknown message IDs are ignored.

### Example: Embed Message

```json
{
  "content": "",
  "embeds": [
    {
      "title": "Welcome to Nebula Official",
      "description": "Welcome @nova. You are member 5.",
      "url": "https://nebulaapp.space",
      "color": "#c084fc"
    }
  ]
}
```

### Response

```json
{
  "ok": true,
  "message": {
    "id": "message-id",
    "channel_id": "channel-id",
    "content": "",
    "embeds": [
      {
        "title": "Welcome to Nebula Official",
        "description": "Welcome @nova. You are member 5.",
        "url": "https://nebulaapp.space",
        "color": 12615932
      }
    ]
  }
}
```

The API automatically cuts message content to 2,000 characters.

Messages sent by bots display a BOT tag.

Bots do not receive their own messages as message events.

### Errors

| Status | Code | Description |
|---|---|---|
| 400 | `invalid_request` | Empty message or no usable embeds. |
| 403 | `missing_permission` | Bot cannot access or send messages in the channel. |
| 404 | `not_found` | Unknown channel or voice channel. |
| 429 | `rate_limited` | Bot message limit or channel slowmode exceeded. |

---

# Account API

The Account API manages bot creation, configuration, and server membership.

Unlike the Bot HTTP API, these endpoints use your signed-in Nebula account session.

**Base URL**

```text
https://scowling-anthill-civil.ngrok-free.dev/api
```

**Authentication**

```http
Authorization: Bearer YOUR_SESSION
```

These endpoints are used internally by:

- Settings → Developer.
- Server Settings → Bots.

A bot application should use the Bot HTTP API instead.

## GET /me/bots

Returns bots owned by the authenticated account.

Request:

```http
GET /me/bots
Authorization: Bearer YOUR_SESSION
```

Response:

```json
{
  "ok": true,
  "limit": 2,
  "bots": [
    {
      "id": "bot-id",
      "name": "Welcome",
      "username": "welcome",
      "displayName": "Welcome",
      "bio": "",
      "avatarUrl": "",
      "tokenTail": "ubKDMs",
      "createdAt": "2026-11-20T05:00:00.000Z",
      "servers": [
        {
          "id": "server-id",
          "name": "Nebula Official"
        }
      ],
      "commands": [
        {
          "name": "ping",
          "description": "Is the bot alive?"
        }
      ]
    }
  ]
}
```

The full bot token is never included.

`tokenTail` contains the last six characters of the current token.

## POST /me/bots

Creates a new bot.

Request:

```http
POST /me/bots
Authorization: Bearer YOUR_SESSION
Content-Type: application/json
```

Body:

```json
{
  "name": "Welcome"
}
```

Parameters:

| Field | Type | Limit |
|---|---|---|
| `name` | string | 2–32 characters |

Response:

```json
{
  "ok": true,
  "bot": {
    "id": "bot-id",
    "name": "Welcome",
    "token": "nb_YOUR_GENERATED_TOKEN"
  }
}
```

The generated token is returned only once.

**Errors**

| Status | Description |
|---|---|
| 400 | Name is shorter than two characters or account already owns two bots. |

## PATCH /me/bots/:id

Updates a bot owned by the authenticated account.

Request:

```http
PATCH /me/bots/BOT_ID
Authorization: Bearer YOUR_SESSION
Content-Type: application/json
```

Body:

```json
{
  "displayName": "Welcome",
  "bio": "Greets new members.",
  "avatarUrl": "/uploads/picture.png"
}
```

Supported fields:

| Field | Description |
|---|---|
| `name` | Sets the display name. |
| `displayName` | Alternative display name field. |
| `bio` | Updates the bot biography. |
| `avatarUrl` | Uploaded Nebula path or HTTP/HTTPS URL. |

## POST /me/bots/:id/token

Generates a new authentication token.

Request:

```http
POST /me/bots/BOT_ID/token
Authorization: Bearer YOUR_SESSION
```

Response:

```json
{
  "ok": true,
  "bot": {
    "id": "bot-id",
    "token": "nb_NEW_GENERATED_TOKEN"
  }
}
```

**Important:** Generating a new token immediately invalidates the previous token.

The new token is returned only once.

## POST /me/bots/:id/test

Queues a test event for a running bot.

Request:

```http
POST /me/bots/BOT_ID/test
Authorization: Bearer YOUR_SESSION
```

Response:

```json
{
  "ok": true
}
```

The bot receives an event with the following data:

```json
{
  "object": "test",
  "bot": "bot-id",
  "by": "account-id",
  "at": "2026-11-20T05:00:00.000Z",
  "hello": "nebula"
}
```

## DELETE /me/bots/:id

Deletes a bot owned by the authenticated account.

Request:

```http
DELETE /me/bots/BOT_ID
Authorization: Bearer YOUR_SESSION
```

Response:

```json
{
  "ok": true
}
```

Deleting a bot:

- Removes it from every server.
- Invalidates its authentication token.
- Deletes the bot account.

## POST /me/bots/:id/servers

Adds an owned bot to a server.

The authenticated account must own the server or have Manage Server permission.

Request:

```http
POST /me/bots/BOT_ID/servers
Authorization: Bearer YOUR_SESSION
Content-Type: application/json
```

Body:

```json
{
  "serverId": "server-id"
}
```

The bot receives a `server_add` event.

**Errors**

| Status | Description |
|---|---|
| 400 | Bot is already in the server or the server has 25 bots. |
| 403 | Account does not have Manage Server permission. |
| 404 | Server does not exist or bot is not owned by the account. |

## DELETE /me/bots/:id/servers/:serverId

Removes an owned bot from a server.

Request:

```http
DELETE /me/bots/BOT_ID/servers/SERVER_ID
Authorization: Bearer YOUR_SESSION
```

The bot receives a `server_remove` event.

## GET /servers/:id/bots

Returns bots in a server that the authenticated account manages, together with bots owned by the account.

Request:

```http
GET /servers/SERVER_ID/bots
Authorization: Bearer YOUR_SESSION
```

Response:

```json
{
  "ok": true,
  "bots": [],
  "mine": []
}
```

`bots` contains server bots, while `mine` contains the account's own bots.

## POST /servers/:id/bots

Adds an owned bot to a server from the server management side.

Request:

```http
POST /servers/SERVER_ID/bots
Authorization: Bearer YOUR_SESSION
Content-Type: application/json
```

Body:

```json
{
  "botId": "bot-id"
}
```

The bot must belong to the authenticated account.

## DELETE /servers/:id/bots/:botId

Removes a bot from a server.

Request:

```http
DELETE /servers/SERVER_ID/bots/BOT_ID
Authorization: Bearer YOUR_SESSION
```

The authenticated account must have Manage Server permission.

The bot does not need to be owned by the account performing the removal.

---

# Events Reference

Nebula bots use long polling to receive events.

Events are generated when activity occurs in servers where the bot has access.

Supported events:

| Event | Description |
|---|---|
| `message` | A person sends a message in a visible channel. |
| `member_join` | A person joins a server. |
| `server_add` | The bot is added to a server. |
| `server_remove` | The bot is removed from a server. |
| `test` | A test event is sent through Developer Settings. |

## Message Event

Triggered when a person sends a message in a channel the bot can see.

Bot messages, webhook messages, and sealed messages are skipped.

A matching registered command is processed after the message listeners.

| Field | Type | Description |
|---|---|---|
| `object` | string | Always `message`. |
| `id` | string | Message ID. |
| `channel_id` | string | Channel ID. |
| `channel_name` | string | Channel name. |
| `server_id` | string | Server ID. |
| `author_id` | string | Author account ID. |
| `author` | string | Author display name. |
| `content` | string | Full message content. |
| `created_at` | string | ISO timestamp. |
| `mentioned` | boolean | Whether the bot was mentioned. |
| `author_bot` | boolean | Always `false`. |
| `command` | string | Registered command name, when matched. |
| `args` | string | Text following the command name. |

### Mention Detection

The `mentioned` field is true when the message contains:

```text
<@BOT_ID>
```

### Command Detection

A message is recognized as a command when it starts with a registered command using `/` or `!`.

Example:

```text
/ping hello
```

The resulting message properties include:

```javascript
msg.command // "ping"
msg.args    // "hello"
msg.argv    // ["hello"]
```

## Member Join Event

Triggered when a person joins a server containing the bot.

Other bots joining do not trigger this event.

The joining bot does not receive this event for itself.

| Field | Type | Description |
|---|---|---|
| `object` | string | Always `member`. |
| `server_id` | string | Server ID. |
| `server_name` | string | Server name. |
| `user_id` | string | Joining user's account ID. |
| `username` | string | Username without `@`. |
| `display_name` | string | Display name. |
| `member_count` | number | Member count after joining, including bots. |

Example:

```javascript
bot.on('member_join', (event) => {
  console.log(
    event.username,
    'joined',
    event.server_name
  );
});
```

## Server Add Event

Triggered when the bot is added to a server.

| Field | Type | Description |
|---|---|---|
| `object` | string | Always `server`. |
| `server_id` | string | Server ID. |
| `name` | string | Server name. |
| `member_count` | number | Member count when added. |

Note: This event uses `name`, not `server_name`.

## Server Remove Event

Triggered when the bot is removed from a server.

| Field | Type | Description |
|---|---|---|
| `object` | string | Always `server`. |
| `server_id` | string | Server ID. |
| `name` | string | Server name. |

## Test Event

Triggered through:

- Settings → Developer → Send test event.
- `POST /api/me/bots/:id/test`.

| Field | Type | Description |
|---|---|---|
| `object` | string | Always `test`. |
| `bot` | string | Bot ID. |
| `by` | string | Account ID that triggered the event. |
| `at` | string | ISO timestamp. |
| `hello` | string | Always `nebula`. |

Example:

```javascript
bot.on('test', (event) => {
  console.log('Test received:', event.hello);
});
```

---

# Examples

## Embed Message

Embeds can be sent using either `bot.send()` or `msg.reply()`.

Pass an empty string when you do not want text above the embed.

```javascript
await bot.send(channelId, 'Scoreboard', {
  embeds: [
    {
      title: 'Match result',
      description: 'Blue won 3-1.',
      url: 'https://nebulaapp.space',
      color: '#7c5cff'
    }
  ]
});
```

## Welcome Bot

This example creates a welcome bot that sends an embed whenever someone joins a server.

It searches for a channel named:

1. `welcome`
2. `general`
3. `lobby`

If none exists, it uses the first visible text channel.

Save as:

```text
welcome.js
```

```javascript
import { Bot } from './nebula.js';

const bot = new Bot('nb_PASTE_YOUR_TOKEN');
const rooms = new Map();

async function welcomeChannel(serverId) {
  if (rooms.has(serverId)) {
    return rooms.get(serverId);
  }

  const data = await bot.api(
    'GET',
    '/servers/' + encodeURIComponent(serverId) + '/channels?type=text'
  );

  const list = data.channels || [];

  const named = list.find((ch) =>
    /^(welcome|general|lobby)$/i.test(ch.name)
  );

  const channel = named || list[0];

  if (!channel) return '';

  rooms.set(serverId, channel.id);

  return channel.id;
}

bot.command(
  'setwelcome',
  'Welcome new people in this channel',
  async (msg) => {
    rooms.set(msg.server_id, msg.channel_id);

    await msg.reply(
      'I will welcome new people in this channel.'
    );
  }
);

bot.on('member_join', async (ev) => {
  if (!ev.server_id) return;

  if (ev.user_id === (bot.me && bot.me.id)) {
    return;
  }

  const channelId = await welcomeChannel(ev.server_id);

  if (!channelId) return;

  const name = ev.username
    ? '@' + ev.username
    : (ev.display_name || 'someone new');

  const count = ev.member_count
    ? ' You are member ' + ev.member_count + '.'
    : '';

  await bot.send(channelId, '', {
    embeds: [
      {
        title: 'Welcome to ' + (ev.server_name || 'the server'),
        description: 'Welcome ' + name + '.' + count,
        color: '#c084fc'
      }
    ]
  });
});

bot.on('ready', (me) => {
  console.log('welcome bot online as', me.username);
});

bot.on('error', (err) => {
  console.error(err);
});

bot.start();
```

**Note:** The `/setwelcome` selection is stored in memory and lasts only until the bot process stops. The example does not persist channel selections between restarts.

## Sending Messages with cURL

You can use the Bot HTTP API without the JavaScript SDK.

Example:

```bash
curl -X POST \
  'https://scowling-anthill-civil.ngrok-free.dev/api/bot/v1/channels/CHANNEL_ID/messages' \
  -H 'Authorization: Bot nb_PASTE_YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "",
    "embeds": [
      {
        "title": "Welcome to Nebula Official",
        "description": "Welcome @nova. You are member 5.",
        "color": "#c084fc"
      }
    ]
  }'
```

Replace:

- `CHANNEL_ID` with the destination channel ID.
- `nb_PASTE_YOUR_TOKEN` with your bot token.

---

# Errors and Limits

## HTTP Status Codes

| Status | Code | Description |
|---|---|---|
| 400 | `invalid_request` | Request body or command list was rejected. |
| 401 | `unauthorized` | Missing or invalid bot token. |
| 403 | `missing_permission` | Bot cannot access or send messages in the channel. |
| 404 | `not_found` | Server, channel, or bot was not found or is not accessible. |
| 413 | `too_large` | Avatar exceeds the maximum file size. |
| 429 | `rate_limited` | Message rate limit or channel slowmode exceeded. |
| 502 | `unavailable` | Avatar could not be stored. |

### Error Response

Bot API errors use the following JSON structure:

```json
{
  "error": "The bot cannot send messages there",
  "code": "missing_permission"
}
```

Applications should check the HTTP status and error code when handling failed requests.

Example:

```javascript
try {
  await bot.send(channelId, 'Hello!');
} catch (err) {
  console.error('Status:', err.status);
  console.error('Code:', err.code);
  console.error('Message:', err.message);
}
```

Account API authentication errors may return a plain `error` message without a `code` field.

## API Limits

| Resource | Limit |
|---|---:|
| Bots per account | 2 |
| Bots per server | 25 |
| Commands per bot | 100 |
| Command name length | 1–32 characters |
| Messages per bot | 30 per minute |
| Message length | 2,000 characters |
| Embed title | 256 characters |
| Embed description | 2,000 characters |
| Embed URL | 500 characters |
| Embeds per message | 4 |
| Bot biography | 300 characters |
| Display name | 32 characters |
| Avatar size | 8 MB |
| Avatar formats | PNG, JPG, GIF, WebP |
| Messages stored per channel | 500 newest messages |
| Events per poll | 100 |
| Maximum event poll wait | 25 seconds |
| Maximum channel slowmode | 21,600 seconds |

### Event Retention

Events are stored in server memory.

- The newest few thousand events are retained.
- Events are cleared when the server restarts.
- Event IDs restart after a server restart.
- Offline bots can only catch up on events that are still retained.
- Each poll returns a maximum of 100 events.

If a server restarts, previously stored `after` values should be treated as `0`.

---

# Security

Bot tokens provide direct access to the bot account.

Anyone with a valid bot token can authenticate as that bot and perform actions allowed by its permissions.

Follow these security recommendations:

- Never publish your bot token in a public GitHub repository.
- Never commit tokens to source code.
- Store tokens in environment variables.
- Do not share tokens through Discord or public messages.
- Generate a new token immediately if an existing token is compromised.
- Use separate tokens for separate bot applications.
- Keep your bot SDK and dependencies under version control.

### Environment Variables

Instead of hardcoding your token:

```javascript
const bot = new Bot('nb_PASTE_YOUR_TOKEN');
```

Use an environment variable:

```javascript
const bot = new Bot(process.env.NEBULA_BOT_TOKEN);
```

Set it before starting the bot.

Windows PowerShell:

```powershell
$env:NEBULA_BOT_TOKEN="nb_YOUR_TOKEN"
node starter.js
```

Linux / macOS:

```bash
export NEBULA_BOT_TOKEN="nb_YOUR_TOKEN"
node starter.js
```

The SDK reads `NEBULA_BOT_TOKEN` when no token is supplied to the constructor.

---

# Additional Information

**Nebula Bot API** is designed to provide developers with a simple way to create integrations, automate server tasks, respond to events, and extend the Nebula platform.

The JavaScript SDK provides a convenient interface, while the HTTP API allows developers to build integrations using any programming language capable of making HTTPS requests.

| Component | Version |
|---|---|
| Nebula Bot API | 1.0.0 |
| JavaScript SDK | 1.0.0 |
| Minimum Node.js | 18+ |

**Official Platform:** https://nebulaapp.space

**Bot API:** https://scowling-anthill-civil.ngrok-free.dev/api/bot/v1

**Documentation:** Nebula Bot API 1.0.0

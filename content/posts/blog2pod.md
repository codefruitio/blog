---
title: Announcing Blog2pod
date: 2024-10-24T19:12:57-05:00
author: ["Tyler Plesetz"]
description: Setup guide for blog2pod
summary: Setup guide for blog2pod
tags: [markdown, code]
categories: [guides]
ShowToc: true
TocOpen: true
weight: 1
---

# blog2pod

## What it does
blog2pod is a self-hosted application that converts web articles and blogs to audio files using AI-based Text-to-Speech (TTS) technology, and then publishes the resulting MP3 files as a podcast feed for you to consume in your podcast player of your choice!

The main reason I created b2p is to give myself a hands free option for listening to articles in the car. My favorite "read it later" app is [Reeder](https://reederapp.com), which doesn't offer an option to listen to the articles in your feed, and the apps that do have that capability, don't integrate with CarPlay or Android Auto for easy selection and playback

blog2pod brings the articles you want to listen to right to your preferred podcast player, which have been designed specifically for listening to spoken-word audio, so they have all the controls you might want. Voice boost, silence skipping, playback speed adjustments, etc..

## How it works
The "frontend" of blog2pod is a [Discord Bot](#create-a-discord-bot). To send an article to b2p for processing, you just need to send the url as a message in a channel where the bot is active, prepended with !blog2pod. For example:

<img src="https://i.imgur.com/JxiS0mD.png" alt="Image" width="500"/>


The simplest way to do this consistently is to set up a shortcut on iOS (or the Android equivalent) to share the URL through a Discord Webhook. Here is my [shortcut template](https://www.icloud.com/shortcuts/b68364e6e3874254b10f231a3b68a17e) for iOS. This gives you the ability to send articles to b2p right from your device's built in share sheet. Detailed instructions on all of this can be found below.

<img src="https://i.imgur.com/Q9wsq5C.png" alt="Image" width="300"/>

Once the url is shared, b2p will generate the audio using the AI platform you configured during setup (Today, b2p supports OpenAI, [ElevenLabs](https://elevenlabs.io/?from=tylerplesetz8843), and Azure OpenAI) and publish it to your podcast feed

<img src="https://i.imgur.com/gZ4m6GE.png" alt="Overcast on iOS" width="300"/>

## Prerequisites
Before starting, make sure you have:
- Discord Server that you manage (So you can add a bot)
- Docker (to run the blog2pod backend)
- API Key for OpenAI, Azure OpenAI, or ElevenLabs (So articles can be converted to audio)

## Setup

### Create a Discord Bot
#### Step 1: Create a Discord Application
1. Go to the [Discord Developer Portal](https://discord.com/developers/applications).
2. Log in with your Discord account if you haven't already.
3. Click the **"New Application"** button in the top-right corner.
4. In the popup window:
   - Enter a name for your application (this will be the bot's name).
   - Click **"Create"**.
5. You will now be redirected to the application's settings page.

#### Step 2: Create a Bot for Your Application
1. On the left sidebar, click the **"Bot"** tab.
2. Click the **"Add Bot"** button.
   - A confirmation popup will appear. Click **"Yes, do it!"** to confirm.
3. Your bot is now created, and you'll see some settings for your bot.

#### Step 3: Customize the Bot Settings (Optional)
1. **Bot Icon**: You can upload an avatar for your bot by clicking on the bot icon image.
2. **Username**: By default, the bot's username will match the application name. You can change this by clicking the name.
3. **Public Bot**: You can control whether other people can invite your bot to their servers by toggling the "Public Bot" option.

#### Step 4: Collect the Bot Token
1. In the **"Bot"** section of your application, you'll see a section labeled **"Token"**.
   - Click the **"Reset Token"** button to generate a new token.
   - **Warning**: Keep this token secret, as anyone with it can control your bot.
2. After generating the token, click the **"Copy"** button to save it. You will need this token later to authenticate the bot when running it.

#### Step 5: Assign Permissions and Invite the Bot to a Server
1. On the left sidebar, click **"OAuth2"**.
2. Under the **"OAuth2 URL Generator"**:
   - In the **"Scopes"** section, select the **"bot"** checkbox.
   - In the **"Bot Permissions"** section, select the permissions your bot will need, such as:
     - `VIEW_CHANNELS`
     - `SEND_MESSAGES`
     - `MANAGE_MESSAGES`
     - `READ_MESSAGE_HISTORY`
     - `MENTION_EVERYONE`
     - `USE_SLASH_COMMANDS`
3. At the bottom of the page, a **"Generated URL"** will appear. Copy this URL.
4. Open a new browser tab and paste the URL into the address bar. This will take you to a page where you can invite the bot to a server.
5. Choose a server where you have administrative privileges and click **"Authorize"**.

> **Note**: Make sure to keep the token private and avoid sharing it publicly.

### Docker Compose

#### Docker Links
[blog2pod](https://hub.docker.com/r/tylerplesetz/blog2pod)

[b2pserve](https://hub.docker.com/r/tylerplesetz/b2pserve)

#### Image Tags
| Docker Image Tag                    | AI Platform     |  Source Code    |
|-------------------------------------|-----------------|-----------------|
| `tylerplesetz/blog2pod:latest`      | **Azure OpenAI** | [Source](https://github.com/blog2pod/blog2pod) |
| `tylerplesetz/blog2pod:oai`         | **OpenAI**       | [Source](https://github.com/blog2pod/blog2pod_oai) |
| `tylerplesetz/blog2pod:el`          | **ElevenLabs**   | [Source](https://github.com/blog2pod/blog2pod_el) |


```yaml
version: "3.3"
services:
  blog2pod:
    container_name: blog2pod
    environment:
      - TTS_VOICE=shimmer
      - DISCORD_BOT_TOKEN=<TOKEN>
      - API_KEY=<APIKEY>
      - TTS_DEPLOYMENT=<azure model deployment name> #Optional - Use for Azure OpenAI (:latest)
      - AZURE_ENDPOINT=<azure endpoint url> #Optional - Use for Azure OpenAI (:latest)
    restart: unless-stopped
    volumes:
      - /path/to/podcasts/directory:/blog2pod/completed
    image: tylerplesetz/blog2pod:oai

  b2pserve:
    container_name: b2pserve
    environment:
      - BASEURL=<url>
      - POD_TITLE=blog2pod
      - POD_DESCRIPTION="AI generated audio for articles that I don't have time to read ..."
      - POD_IMAGE=https://i.imgur.com/y2WVeWh.png
      - POD_AUTHOR=codefruit
    restart: unless-stopped
    volumes:
      - /path/to/podcasts/directory:/b2pserve/completed
    ports:
      - "8000:8000"
    image: tylerplesetz/b2pserve:latest
networks: {}
```

#### Environment variables

| Name                | Container | Required?                        | Notes                                                                                                                                           |
| ------------------- | --------- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `TTS_VOICE`         | blog2pod  | Yes                              | Used to determine voice used by selected AI platform<br>- OpenAI uses names like "shimmer"<br>- ElevenLabs uses IDs like "XrExE9yKIg1WjnnlVkGX" |
| `API_KEY`           | blog2pod  | Yes                              | API key for your selected AI platform                                                                                                           |
| `DISCORD_BOT_TOKEN` | blog2pod  | Yes                              | Token for your Discord Bot                                                                                                                      |
| `TTS_DEPLOYMENT`    | blog2pod  | latest: yes<br>oai: no<br>el: no | If you're using Azure OpenAI, you'll know what this is                                                                                          |
| `AZURE_ENDPOINT`    | blog2pod  | latest: yes<br>oai: no<br>el: no | If you're using Azure OpenAI, you'll know what this is                                                                                          |
| `BASEURL`           | b2pserve  | Yes                              | Public URL for accessing your podcast feed                                                                                                      |
| `POD_TITLE`         | b2pserve  | Yes                              | Title of your published podcast                                                                                                                 |
| `POD_DESCRIPTION`   | b2pserve  | Yes                              | Description of your published podcast                                                                                                           |
| `POD_IMAGE`         | b2pserve  | Yes                              | Artwork for your published podcast                                                                                                              |
| `POD_AUTHOR`        | b2pserve  | Yes                              | Author of your published podcast                                                                                                                |

Once you've set up Discord and Docker, you are ready to use blog2pod. Keep reading for optional configurations.

## Optional Configurations

### iOS Shortcut Setup (Optional, but recommended)

#### Create a Discord Webhook
##### Step 1: Open Discord and Select a Server
1. Open Discord and navigate to the server where you want to create the webhook.
2. You must have the necessary permissions (typically, **Manage Webhooks** or **Administrator**) to create a webhook in the server.

##### Step 2: Go to Server Settings
1. Click the server name at the top of the channel list to open the server dropdown menu.
2. Select **"Server Settings"**.

##### Step 3: Create a Webhook
1. In the left sidebar under the **"Settings"** menu, click **"Integrations"**.
2. On the **"Integrations"** page, you'll see a section for **Webhooks**.
3. Click the **"Create Webhook"** button.

##### Step 4: Customize the Webhook
1. **Webhook Name**: Enter a name for your webhook. This is the name that will appear when the webhook sends messages.
2. **Webhook Icon**: You can upload an image to use as the webhook’s avatar.
3. **Channel**: Choose the channel where the webhook will post messages. This is the channel that the webhook will send data to when triggered.
4. Once you've configured the webhook, click **"Copy Webhook URL"** to copy the webhook URL. This URL is what you'll use to send data to Discord.

##### Step 5: Save Webhook Settings
1. After copying the webhook URL, click **"Save"** to save your new webhook.

#### Configure the iOS Shortcut
1. Install the Shortcut using [this link](https://www.icloud.com/shortcuts/b68364e6e3874254b10f231a3b68a17e)
2. Enter your Discord Webhook URL

<img src="https://i.imgur.com/ZxWa5jP.png" alt="Image" width="300"/>

## Pre-release disclaimer

This is pre-release software and anything can break at any time. I make no guarantees about the stability of this software, forward-compatibility of updates, or integrity (both related to and independent of blog2pod). Essentially, use at your own risk and expect there will be rough edges for now.

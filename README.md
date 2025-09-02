# @ngtongsheng/n8n-nodes-slack-socket-mode-pubsub-trigger

<div align="center">
  <img src="https://raw.githubusercontent.com/n8n-io/n8n/master/assets/n8n-logo.png" width="200" alt="n8n logo">
  <h3>+</h3>
  <img src="https://cdn.worldvectorlogo.com/logos/slack-new-logo.svg" width="100" alt="Slack logo">
</div>

This is an n8n community node that enables real-time Slack event processing using Socket Mode with an intelligent publish-subscribe pattern. It allows you to listen to Slack events in real-time without requiring public URLs for webhooks, making it perfect for local development and secure environments.

**Key Features:**
- 🚀 **Real-time event processing** via Slack Socket Mode
- 🔄 **Intelligent pub-sub pattern** prevents message loss with multiple connections
- 🛡️ **No webhook URLs required** - perfect for local development and secure environments
- ⚡ **Connection pooling** for optimal resource usage
- 🎯 **Advanced filtering** with regex pattern matching
- 📱 **Multiple event types** - messages, mentions, reactions, and more

[n8n](https://n8n.io/) is a [fair-code licensed](https://docs.n8n.io/reference/license/) workflow automation platform.

## Table of Contents
- [Installation](#installation)
- [Slack App Setup](#slack-app-setup)
- [Credentials](#credentials)
- [Node Configuration](#node-configuration)
- [Supported Events](#supported-events)
- [Advanced Features](#advanced-features)
- [Usage Examples](#usage-examples)
- [Development](#development)
- [Compatibility](#compatibility)
- [Resources](#resources)
- [License](#license)

## Installation

### Community Installation

Follow the [installation guide](https://docs.n8n.io/integrations/community-nodes/installation/) in the n8n community nodes documentation.

### Manual Installation

Using Bun (recommended):
```bash
bun add @ngtongsheng/n8n-nodes-slack-socket-mode-pubsub-trigger
```

Using npm:
```bash
npm install @ngtongsheng/n8n-nodes-slack-socket-mode-pubsub-trigger
```

Using pnpm:
```bash
pnpm add @ngtongsheng/n8n-nodes-slack-socket-mode-pubsub-trigger
```

After installation, restart n8n and the "Slack Socket Mode Trigger" node will be available in the trigger nodes section.

## Slack App Setup

Before using this node, you need to create a Slack app with Socket Mode enabled:

1. **Create a Slack App**
   - Go to [https://api.slack.com/apps](https://api.slack.com/apps)
   - Click "Create New App" → "From scratch"
   - Enter your app name and select your workspace

2. **Enable Socket Mode**
   - Navigate to "Socket Mode" in the left sidebar
   - Toggle "Enable Socket Mode" to ON
   - Generate an "App-Level Token" with the `connections:write` scope
   - Save the token (starts with `xapp-`)

3. **Configure OAuth & Permissions**
   - Go to "OAuth & Permissions" in the left sidebar
   - Add the following Bot Token Scopes:
     - `app_mentions:read` - To receive app mentions
     - `channels:history` - To read message history
     - `channels:read` - To access channel information
     - `chat:write` - To send messages (if needed for responses)
     - `reactions:read` - To read emoji reactions
     - `team:read` - To access workspace information
   - Install the app to your workspace
   - Copy the "Bot User OAuth Token" (starts with `xoxb-`)

4. **Get Signing Secret**
   - Go to "Basic Information" in the left sidebar
   - Copy the "Signing Secret" from the App Credentials section

5. **Configure Event Subscriptions**
   - Go to "Event Subscriptions" in the left sidebar
   - Enable "Socket Mode" (should already be enabled)
   - Subscribe to bot events you want to listen to:
     - `app_mention` - When your bot is mentioned
     - `message.channels` - Messages in public channels
     - `message.groups` - Messages in private channels
     - `reaction_added` - When reactions are added to messages

6. **Configure Interactive Components (for Action events)**
   - Go to "Interactivity & Shortcuts" in the left sidebar
   - Enable "Interactivity"
   - Socket Mode will handle the interactions automatically
   - No Request URL needed when using Socket Mode

## Credentials

Create a new credential of type "Slack Socket Mode Credential" with:

- **Signing Secret**: Your Slack App Signing Secret from Basic Information
- **Bot User OAuth Token**: Your Bot User OAuth Token (starts with `xoxb-`)
- **App-Level Token**: Your App-Level Token for Socket Mode (starts with `xapp-`)

## Node Configuration

The Slack Socket Mode Trigger node provides the following configuration options:

### Trigger Events
- **Message**: Triggers when messages are sent to channels
- **App Mention**: Triggers when your bot is mentioned (@your_bot_name)
- **Reaction Added**: Triggers when emoji reactions are added to messages
- **Action**: Triggers when interactive actions are performed (buttons, select menus, etc.)

### Channel Filtering
- **Channels to Watch**: Select specific channels to monitor
- If no channels are selected, all accessible channels will be monitored
- Supports both public and private channels (based on bot permissions)

### Message Filtering (Message events only)
- **Message Filter**: Optional regex pattern to filter messages
- **Allow Bot Messages**: Include/exclude messages from bots and message updates
- By default, bot messages and message changes are filtered out

### Action Filtering (Action events only)
- **Action ID**: Optional action ID to filter specific interactive actions
- Leave empty to listen to all action types
- Useful for targeting specific buttons or interactive components

## Supported Events

### Message Events
- Triggers on new messages in monitored channels
- Supports regex pattern matching for content filtering
- Option to include/exclude bot messages
- Filters out message updates and changes by default

### App Mention Events
- Triggers when your bot is mentioned in any message
- Works across all channels where the bot has access
- Includes the full message context and mention details

### Reaction Added Events
- Triggers when emoji reactions are added to messages
- Provides reaction details including emoji type and user
- Works on messages in monitored channels

### Action Events
- Triggers when users interact with buttons, select menus, or other interactive components
- Supports filtering by specific Action ID for targeted responses
- Provides full action context including user, channel, and interaction details
- Works with Slack's Block Kit interactive elements

## Advanced Features

### Intelligent Connection Management
- **Connection Pooling**: Reuses Socket Mode connections across multiple workflow instances
- **Pub-Sub Pattern**: Prevents message loss when multiple workflows use the same bot token
- **Automatic Cleanup**: Removes unused connections when workflows are deactivated
- **Error Handling**: Robust error handling with automatic reconnection

### Why Pub-Sub Pattern?
When multiple Socket Mode connections are established with the same bot token, Slack distributes events across connections in a round-robin fashion. This means:
- ❌ **Without pub-sub**: 3 workflows = 3 connections = each receives only ~33% of messages
- ✅ **With pub-sub**: 3 workflows = 1 shared connection = all workflows receive 100% of messages

The pub-sub pattern ensures **guaranteed message delivery** to all subscribers while optimizing resource usage.

### Performance Optimizations
- **Regex Caching**: Compiled regex patterns are cached for better performance
- **Event Filtering**: Early filtering at the Socket Mode level reduces processing overhead
- **Subscriber Pattern**: Efficient pub-sub pattern for handling multiple workflow triggers

### Channel Management
- **Dynamic Channel Loading**: Automatically fetches available channels for selection
- **Permission Awareness**: Only shows channels the bot has access to
- **Real-time Updates**: Reflects current channel permissions and availability

## Usage Examples

### 1. Auto-Responder Bot
```
[Slack Socket Mode Trigger] → [Switch] → [Slack]
```
- **Trigger**: Message events with regex pattern `help|support`
- **Switch**: Route based on message content
- **Slack**: Send appropriate help responses

### 2. Mention Handler
```
[Slack Socket Mode Trigger] → [HTTP Request] → [Slack]
```
- **Trigger**: App mention events
- **HTTP Request**: Fetch data from external API
- **Slack**: Reply with fetched information

### 3. Reaction Monitor
```
[Slack Socket Mode Trigger] → [Code] → [Database]
```
- **Trigger**: Reaction added events
- **Code**: Process reaction data
- **Database**: Store reaction analytics

### 4. Channel-Specific Automation
```
[Slack Socket Mode Trigger] → [IF] → [Multiple Actions]
```
- **Trigger**: Messages in specific channels only
- **IF**: Check message content or user
- **Actions**: Perform different actions based on conditions

### 5. Interactive Button Handler
```
[Slack Socket Mode Trigger] → [Switch] → [Slack]
```
- **Trigger**: Action events with specific Action ID filtering
- **Switch**: Route based on button action ID or interaction type
- **Slack**: Send contextual responses or update messages

## Development

This project uses [Bun](https://bun.sh) as its package manager for improved performance.

### Setup
```bash
# Install Bun
curl -fsSL https://bun.sh/install | bash

# Install dependencies
bun install

# Build the project
bun run build

# Development mode with watch
bun run dev
```

### Scripts
- `bun run build` - Build the project and copy assets
- `bun run dev` - Development mode with TypeScript watch
- `bun run format` - Format code with Biome
- `bun run lint` - Lint code with Biome
- `bun run check` - Run all checks (format + lint)

### Project Structure
```
├── credentials/
│   ├── SlackSocketModeCredential.credentials.ts
│   └── assets/
├── nodes/
│   └── SlackSocketModeTrigger/
│       ├── SlackSocketModeTrigger.node.ts
│       └── assets/
└── package.json
```

## Compatibility

- **n8n**: Requires version 1.17.0 or later
- **Node.js**: Requires version 18.10 or later
- **Bun**: Requires version 1.0.0 or later (for development)
- **Slack API**: Uses @slack/bolt v4.4.0

## Resources

- [GitHub Repository](https://github.com/ngtongsheng/n8n-nodes-slack-socket-mode-pubsub-trigger)
- [n8n Community Nodes Documentation](https://docs.n8n.io/integrations/community-nodes/)
- [Slack API Documentation](https://api.slack.com/apis)
- [Slack Socket Mode Documentation](https://api.slack.com/apis/connections/socket)
- [Slack Events API Documentation](https://api.slack.com/events)
- [Slack Bolt Framework](https://slack.dev/bolt-js/concepts)
- [Bun Documentation](https://bun.sh/docs)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

## Support

If you encounter any issues or have questions:
1. Check the [GitHub Issues](https://github.com/ngtongsheng/n8n-nodes-slack-socket-mode-pubsub-trigger/issues)
2. Create a new issue with detailed information about your problem
3. Include your n8n version, Node.js version, and error messages

## License

MIT License

Copyright (c) 2025 Ng Tong Sheng

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

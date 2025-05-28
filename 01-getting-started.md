# Getting Started with Tsuki Bot

This guide will help you set up and run your first Tsuki Discord bot in just a few steps.

## 📋 Prerequisites

Before you begin, make sure you have:

- **Node.js 18+** installed on your system
- **npm** or **yarn** package manager
- A **Discord Application** with a bot token
- Basic knowledge of **JavaScript/TypeScript**

## 🛠️ Installation

### 1. Clone or Download the Project

```bash
# If using git
git clone <your-repo-url>
cd tsuki-bot

# Or extract the zip file you downloaded
```

### 2. Install Dependencies

```bash
npm install
# or
yarn install
```

### 3. Environment Setup

Create a `.env` file in the root directory:

```bash
cp .env.example .env
```

Edit the `.env` file with your Discord bot credentials:

```env
TOKEN=your_discord_bot_token_here
ERROR_LOGS_WEBHOOK=your_error_webhook_url
GUILD_LOGS_WEBHOOK=your_guild_webhook_url
COMMAND_LOGS_WEBHOOK=your_command_webhook_url
```

## 🤖 Creating a Discord Application

### 1. Discord Developer Portal

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Click **"New Application"**
3. Give your application a name
4. Navigate to the **"Bot"** section
5. Click **"Add Bot"**

### 2. Get Your Bot Token

1. In the Bot section, click **"Reset Token"**
2. Copy the token and paste it in your `.env` file
3. **Keep this token secret!**

### 3. Bot Permissions

In the Bot section, enable these **Privileged Gateway Intents**:
- ✅ **Server Members Intent**
- ✅ **Message Content Intent**

### 4. Invite Your Bot

1. Go to **OAuth2 > URL Generator**
2. Select **Scopes**: `bot` and `applications.commands`
3. Select **Permissions**:
   - Send Messages
   - Use Slash Commands
   - Read Message History
   - View Channels
4. Copy the generated URL and open it to invite your bot

## 🚀 Running the Bot

### Development Mode

```bash
# Build TypeScript files
npm run build

# Run the bot
npm test
```

### Watch Mode (Auto-rebuild)

```bash
# In one terminal - watch for changes
npx tsc --watch

# In another terminal - run the bot
npm test
```

## 📁 Project Structure Overview

```
tsuki-bot/
├── src/
│   ├── classes/          # Core classes (Client, Command, etc.)
│   ├── events/           # Discord event handlers
│   ├── modules/          # Feature modules
│   ├── utilities/        # Helper functions
│   ├── debugger/         # Debug commands
│   ├── client.ts         # Main client setup
│   ├── config.ts         # Bot configuration
│   └── index.ts          # Entry point (sharding)
├── app/                  # Compiled JavaScript (auto-generated)
├── .env                  # Environment variables
└── package.json          # Dependencies and scripts
```

## ✅ Verify Installation

Once your bot is running, you should see:

```
[ SHARDING ] - Launched shard with ID : 0
[ SHARDING ] - Shard 0 ready with X guilds and Y users
Bot is ready! Logged in as YourBotName#1234
```

## 🎯 Your First Command

The bot comes with example commands. Try these in Discord:

```
.example          # Run example command
/example          # Slash command version
```

## 🐛 Common Issues

### Bot Not Responding
- ✅ Check if bot token is correct
- ✅ Verify bot has necessary permissions
- ✅ Ensure Message Content Intent is enabled

### TypeScript Errors
- ✅ Run `npm run build` to compile
- ✅ Check for missing dependencies

### Permission Errors
- ✅ Make sure bot has Send Messages permission
- ✅ Check role hierarchy in your Discord server

## 📚 Next Steps

Now that your bot is running:

1. **[Project Structure](./02-project-structure.md)** - Learn how the code is organized
2. **[Configuration](./03-configuration.md)** - Customize your bot settings
3. **[Commands](./05-commands.md)** - Create your first custom command
4. **[Modules](./08-modules.md)** - Add new features with modules

## 🎉 Congratulations!

Your Tsuki bot is now running! The framework provides a solid foundation for building powerful Discord bots with TypeScript.

---

**Next:** Learn about the [Project Structure](./02-project-structure.md) to understand how everything works together.

# Configuration

Learn how to configure your Tsuki bot for different environments and customize its behavior.

## 📁 Configuration Files

### Environment Variables (`.env`)

The `.env` file contains sensitive information and should never be committed to version control:

```env
# Required - Your Discord bot token
TOKEN=your_bot_token_here

# Optional - Webhook URLs for logging
ERROR_LOGS_WEBHOOK=https://discord.com/api/webhooks/...
GUILD_LOGS_WEBHOOK=https://discord.com/api/webhooks/...
COMMAND_LOGS_WEBHOOK=https://discord.com/api/webhooks/...
```

### Main Configuration (`src/config.ts`)

This file contains all the bot's configuration settings:

```typescript
// Basic Settings
const prefix = ".";                    // Command prefix
const color = "#F59FC5";              // Embed color
const admins = ["user_id_1", "user_id_2"]; // Admin user IDs
```

## ⚙️ Configuration Options

### 1. Basic Settings

```typescript
// Command prefix for text commands
const prefix = ".";

// Default embed color (hex format)
const color = "#F59FC5";

// Bot admin user IDs
const admins = ["692617937512562729", "1141593293466107954"];
```

### 2. Discord Intents

Intents control what events your bot receives:

```typescript
const intents = [
  DJS.GatewayIntentBits.Guilds,         // Guild information
  DJS.GatewayIntentBits.GuildMembers,   // Member updates
  DJS.GatewayIntentBits.GuildMessages,  // Server messages
  DJS.GatewayIntentBits.DirectMessages, // DM messages
  DJS.GatewayIntentBits.MessageContent, // Message content
  DJS.GatewayIntentBits.GuildVoiceStates // Voice events
];
```

**Important:** Some intents require verification for large bots (100+ servers).

### 3. Partials

Partials allow handling of incomplete Discord objects:

```typescript
const partials = [
  DJS.Partials.User,        // Partial user data
  DJS.Partials.Channel,     // Partial channel data
  DJS.Partials.Message,     // Partial message data (needed for old messages)
  DJS.Partials.GuildMember  // Partial member data
];
```

### 4. Bot Presence

Configure how your bot appears online:

```typescript
const presence = {
  status: DJS.PresenceUpdateStatus.Online,  // Online, Idle, DoNotDisturb, Invisible
  activities: [{
    type: DJS.ActivityType.Custom,          // Playing, Streaming, Listening, Watching, Custom
    name: "By ━● 1sT-Services"
  }]
};
```

**Activity Types:**
- `Playing` - "Playing Minecraft"
- `Streaming` - "Streaming on Twitch"
- `Listening` - "Listening to Spotify"
- `Watching` - "Watching YouTube"
- `Custom` - Custom status message

### 5. Allowed Mentions

Control who the bot can mention:

```typescript
const allowedMentions = {
  repliedUser: false,                     // Don't ping replied user
  parse: [
    DJS.AllowedMentionsTypes.User,        // Allow @user mentions
    DJS.AllowedMentionsTypes.Role         // Allow @role mentions
  ]
  // Note: Everyone and Here mentions are disabled by default
};
```

### 6. Cache Options

Optimize memory usage with cache limits:

```typescript
const cacheOptions = {
  ...DJS.Options.DefaultMakeCacheSettings,

  // Message caching
  MessageManager: 110,                    // Cache 110 messages per channel
  DMMessageManager: 10,                   // Cache 10 DM messages
  GuildMessageManager: 100,               // Cache 100 guild messages

  // Voice state caching
  VoiceStateManager: {
    maxSize: 100,
    keepOverLimit: (_) => !_.channelId    // Keep members not in voice
  },

  // User caching
  UserManager: {
    maxSize: 100,
    keepOverLimit: (_) => _.id === _.client.user.id  // Always keep bot user
  },

  // Member caching
  GuildMemberManager: {
    maxSize: 100,
    keepOverLimit: (_) => _.id === _.client.user.id  // Always keep bot member
  }
};
```

## 🔗 Webhook Configuration

Webhooks provide logging to Discord channels:

### Setting Up Webhooks

1. **Create Webhook in Discord:**
   - Go to Channel Settings → Integrations → Webhooks
   - Click "New Webhook"
   - Copy the webhook URL

2. **Add to Environment:**
   ```env
   ERROR_LOGS_WEBHOOK=https://discord.com/api/webhooks/123/abc...
   GUILD_LOGS_WEBHOOK=https://discord.com/api/webhooks/456/def...
   COMMAND_LOGS_WEBHOOK=https://discord.com/api/webhooks/789/ghi...
   ```

### Webhook Types

```typescript
const webhooks = {
  errorLogs: new DJS.WebhookClient({ url: process.env.ERROR_LOGS_WEBHOOK! }),
  guildLogs: new DJS.WebhookClient({ url: process.env.GUILD_LOGS_WEBHOOK! }),
  commandLogs: new DJS.WebhookClient({ url: process.env.COMMAND_LOGS_WEBHOOK! })
};
```

- **errorLogs** - Bot errors and exceptions
- **guildLogs** - Server join/leave events
- **commandLogs** - Command usage tracking

## 🎛️ Customizing Configuration

### Adding New Settings

1. **Add to config.ts:**
   ```typescript
   const myCustomSetting = "custom_value";

   export const config = {
     // ... existing settings
     myCustomSetting
   };
   ```

2. **Use in your code:**
   ```typescript
   // In any file
   import { config } from "../config.js";
   console.log(config.myCustomSetting);

   // In classes with client access
   console.log(this.client.config.myCustomSetting);
   ```

### Environment-Specific Config

Create different configurations for development and production:

```typescript
const isDevelopment = process.env.NODE_ENV === "development";

const config = {
  prefix: isDevelopment ? "dev." : ".",
  color: isDevelopment ? "#FF0000" : "#F59FC5",
  // ... other settings
};
```

## 🛡️ Security Best Practices

### 1. Environment Variables
```bash
# ✅ Good - Use .env file
TOKEN=your_token_here

# ❌ Bad - Never hardcode tokens
const token = "MTI3OT...";
```

### 2. Admin Configuration
```typescript
// ✅ Good - Use user IDs
const admins = ["123456789012345678"];

// ❌ Bad - Use usernames (can change)
const admins = ["username"];
```

### 3. Webhook URLs
```bash
# ✅ Good - Optional webhooks
ERROR_LOGS_WEBHOOK=https://discord.com/api/webhooks/...

# ❌ Bad - Don't expose webhook URLs in code
```

## 🔧 Common Configuration Patterns

### 1. Multi-Environment Setup

```typescript
// config/development.ts
export const devConfig = {
  prefix: "dev.",
  color: "#FF0000",
  logLevel: "debug"
};

// config/production.ts
export const prodConfig = {
  prefix: ".",
  color: "#F59FC5",
  logLevel: "info"
};
```

### 2. Feature Toggles

```typescript
const config = {
  features: {
    enableSlashCommands: true,
    enableTextCommands: true,
    enableModeration: false,
    enableMusic: true
  }
};
```

### 3. Rate Limiting

```typescript
const config = {
  rateLimits: {
    commands: {
      global: 5000,    // 5 seconds global cooldown
      user: 3000,      // 3 seconds per user
      guild: 1000      // 1 second per guild
    }
  }
};
```

## 📊 Configuration Validation

Add validation to ensure configuration is correct:

```typescript
// In config.ts
if (!process.env.TOKEN) {
  throw new Error("TOKEN is required in .env file");
}

if (admins.length === 0) {
  console.warn("No admins configured");
}

if (!color.startsWith("#")) {
  throw new Error("Color must be a valid hex code");
}
```

## 🚀 Performance Optimization

### Cache Optimization

```typescript
// For small bots (< 100 servers)
const cacheOptions = {
  MessageManager: 50,
  UserManager: { maxSize: 50 }
};

// For large bots (1000+ servers)
const cacheOptions = {
  MessageManager: 10,
  UserManager: { maxSize: 200 },
  GuildMemberManager: { maxSize: 500 }
};
```

### Intent Optimization

Only enable intents you actually use:

```typescript
// Minimal intents for slash-only bot
const intents = [
  DJS.GatewayIntentBits.Guilds
];

// Full intents for feature-rich bot
const intents = [
  DJS.GatewayIntentBits.Guilds,
  DJS.GatewayIntentBits.GuildMembers,
  DJS.GatewayIntentBits.GuildMessages,
  DJS.GatewayIntentBits.MessageContent
];
```

---

**Next:** Learn about [Client & Sharding](./04-client-sharding.md) to understand the bot architecture.

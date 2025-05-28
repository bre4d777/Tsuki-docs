# Client & Sharding

Understanding how the Tsuki bot client works and how sharding scales your bot for large deployments.

## 🤖 ExtendedClient Overview

The `ExtendedClient` class extends Discord.js's `Client` with additional functionality:

```typescript
export class ExtendedClient extends DJS.Client<true> {
  // Built-in logger
  log(...args: Parameters<Logger["log"]>) {
    return this.#logger.log(...args);
  }

  // Database access
  db = db;

  // Configuration
  config = config;
  webhooks = config.webhooks;

  // Command storage
  commands = new DJS.Collection<string, BaseCommand>();

  // Utility builders
  get button() { return new ExtendedButtonBuilder(); }
  get embed() { return new DJS.EmbedBuilder().setColor(this.config.color); }
  get row() { return new DJS.ActionRowBuilder<DJS.AnyComponentBuilder>(); }
}
```

## 🏗️ Client Architecture

### 1. Initialization Flow

```
index.ts → ShardingManager → client.ts → ExtendedClient → Module Loading
```

**Step-by-step process:**
1. `index.ts` creates ShardingManager
2. ShardingManager spawns shards
3. Each shard runs `client.ts`
4. `client.ts` creates ExtendedClient
5. Modules and events are loaded

### 2. Client Setup

```typescript
// client.ts
import { ExtendedClient } from "./classes/client.js";
import * as Loaders from "./utilities/loaders.js";

// Load environment variables
process.loadEnvFile();

// Create extended client
const client = new ExtendedClient();

// Load events and modules
await Loaders.loadEvents(client);
await Loaders.loadModules(client);

// Login to Discord
client.login(client.config.token);
```

## ⚡ Sharding System

### What is Sharding?

Sharding splits your bot across multiple processes to handle more Discord servers:

- **Small bots** (< 2,500 servers): No sharding needed
- **Large bots** (2,500+ servers): Sharding required by Discord
- **Huge bots** (10,000+ servers): Multiple shards recommended

### Sharding Manager

```typescript
// index.ts
const sharder = new DJS.ShardingManager(mainFile, {
  token: process.env.token
});

// Handle shard creation
sharder.on("shardCreate", (shard) => {
  logger.log(`Launched shard with ID: ${shard.id}`, "info");

  shard.on("ready", async () => {
    const [guilds, users] = await shard.eval((c) => [
      c.guilds.cache.size,
      c.guilds.cache.map((g) => g.memberCount).reduce((a, b) => a + b, 0)
    ]);

    logger.log(
      `Shard ${shard.id} ready with ${guilds} guilds and ${users} users`,
      "info"
    );
  });
});

// Spawn shards
sharder.spawn({ timeout: -1 });
```

### Shard Communication

Shards can communicate with each other:

```typescript
// Broadcast to all shards
client.shard?.broadcastEval((c) => {
  // Code runs on all shards
  return c.guilds.cache.size;
});

// Get data from specific shard
client.shard?.eval((c) => c.uptime, { shard: 0 });

// Send data between shards
client.shard?.send({ type: 'custom', data: 'hello' });
```

## 🛠️ Client Features

### 1. Built-in Logger

```typescript
// Available logging levels
client.log("Info message", "info");
client.log("Warning message", "warn");
client.log("Error message", "error");
client.log("Debug message", "debug");
client.log("Success message", "success");
```

### 2. Database Integration

```typescript
// Direct database access
const data = client.db.get("user_123");
client.db.set("user_123", { coins: 100 });
client.db.delete("user_123");

// Database methods
client.db.add("user_123.coins", 50);
client.db.subtract("user_123.coins", 25);
client.db.has("user_123");
client.db.all(); // Get all data
```

### 3. Utility Builders

```typescript
// Create embeds with default color
const embed = client.embed
  .setTitle("Example")
  .setDescription("This embed uses the config color");

// Create buttons
const button = client.button
  .setCustomId("example")
  .setLabel("Click me")
  .setStyle(ButtonStyle.Primary);

// Create action rows
const row = client.row.addComponents(button);
```

### 4. Command Management

```typescript
// Access commands collection
const command = client.commands.get("ping");

// Add commands programmatically
client.commands.set("newcommand", new MyCommand());

// List all commands
client.commands.forEach((cmd, name) => {
  console.log(`${name}: ${cmd.description}`);
});
```

## 🔧 Client Configuration

### Constructor Options

```typescript
constructor() {
  super({
    failIfNotExists: false,
    presence: config.presence,
    intents: config.intents,
    partials: config.partials,
    allowedMentions: config.allowedMentions,
    makeCache: DJS.Options.cacheWithLimits(config.cacheOptions)
  });
}
```

### Cache Configuration

```typescript
const cacheOptions = {
  ...DJS.Options.DefaultMakeCacheSettings,
  MessageManager: 110,                    // Cache recent messages
  UserManager: { maxSize: 100 },          // Limit user cache
  GuildMemberManager: { maxSize: 100 }    // Limit member cache
};
```

## 🐛 Debugging Integration

The client includes advanced debugging tools:

```typescript
// Automatically attached debugger
new Debugger(this, {
  prefix: this.config.prefix,
  globalVariables: { Context }
}).attach();
```

**Debug commands available:**
- `.js <code>` - Execute JavaScript
- `.sh <command>` - Run shell commands
- `.curl <url>` - Make HTTP requests
- `.cat <file>` - Read files
- `.rtt` - Test response time

## 🚀 Performance Optimization

### 1. Memory Management

```typescript
// Optimize cache sizes
const cacheOptions = {
  MessageManager: 50,           // Reduce for memory
  UserManager: { maxSize: 100 },
  keepOverLimit: (item) => {
    // Keep important items
    return item.id === client.user.id;
  }
};
```

### 2. Event Optimization

```typescript
// Only listen to needed events
const intents = [
  DJS.GatewayIntentBits.Guilds,         // Essential
  DJS.GatewayIntentBits.GuildMessages   // If needed
  // Don't add unnecessary intents
];
```

### 3. Shard Optimization

```typescript
// Configure shard count
const sharder = new DJS.ShardingManager(mainFile, {
  token: process.env.TOKEN,
  totalShards: 'auto',        // Let Discord decide
  shardArgs: ['--max-old-space-size=2048']  // Increase memory
});
```

## 📊 Monitoring & Statistics

### Shard Statistics

```typescript
// Get shard info
const shardId = client.shard?.ids[0];
const shardCount = client.shard?.count;

// Get shard ping
const ping = client.ws.ping;

// Get shard status
const status = client.ws.status;
```

### Guild Statistics

```typescript
// Total guilds across all shards
const totalGuilds = await client.shard?.broadcastEval(c =>
  c.guilds.cache.size
).then(results => results.reduce((acc, count) => acc + count, 0));

// Total users across all shards
const totalUsers = await client.shard?.broadcastEval(c =>
  c.guilds.cache.reduce((acc, guild) => acc + guild.memberCount, 0)
).then(results => results.reduce((acc, count) => acc + count, 0));
```

## 🔄 Lifecycle Events

### Client Ready

```typescript
client.on('ready', () => {
  console.log(`Bot ready! Logged in as ${client.user.tag}`);

  // Set activity
  client.user.setActivity('with TypeScript', { type: 'PLAYING' });
});
```

### Shard Events

```typescript
client.on('shardReady', (id) => {
  console.log(`Shard ${id} is ready`);
});

client.on('shardDisconnect', (event, id) => {
  console.log(`Shard ${id} disconnected`);
});

client.on('shardReconnecting', (id) => {
  console.log(`Shard ${id} is reconnecting`);
});
```

## 🎯 Best Practices

### 1. Error Handling

```typescript
client.on('error', (error) => {
  client.log(`Client error: ${error.message}`, 'error');
  // Send to webhook if configured
  client.webhooks.errorLogs?.send({
    content: `**Client Error:**\n\`\`\`${error.stack}\`\`\``
  });
});
```

### 2. Graceful Shutdown

```typescript
process.on('SIGINT', () => {
  client.log('Received SIGINT, shutting down gracefully', 'info');
  client.destroy();
  process.exit(0);
});
```

### 3. Resource Cleanup

```typescript
client.on('guildDelete', (guild) => {
  // Clean up guild-specific data
  client.db.delete(`guild_${guild.id}`);
});
```

---

**Next:** Learn about the [Command System](./05-commands.md) to create interactive bot commands.

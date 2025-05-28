# Command System

Learn how to create, manage, and customize commands in the Tsuki bot framework.

## 🎯 Command Basics

Commands in Tsuki are built using the `BaseCommand` class and support both text and slash commands automatically.

### Basic Command Structure

```typescript
import { BaseCommand } from "../../classes/command.js";
import type { Context } from "../../classes/context.js";

export default class extends BaseCommand {
  description = "A simple example command";

  async run(ctx: Context) {
    await ctx.reply("Hello, world!");
  }
}
```

## 🏗️ BaseCommand Class

The `BaseCommand` class provides the foundation for all commands:

```typescript
export abstract class BaseCommand {
  name!: string;                    // Auto-set from filename
  category!: string;                // Auto-set from module folder
  description!: string;             // Required - command description

  // Optional properties
  nsfw = false;                     // NSFW channels only
  requireAdmin = false;             // Admin users only
  usage = "";                       // Usage example
  cooldown = 5000;                  // Cooldown in milliseconds
  aliases = <string[]>[];           // Alternative command names
  examples = <string[]>[];          // Usage examples

  // Slash command options
  slash = true;                     // Enable slash command
  slashOnly = false;                // Slash command only
  slashOptions = [];                // Slash command options

  // Permission requirements
  botPerms = [];                    // Bot permissions needed
  userPerms = [];                   // User permissions needed

  // Required method
  abstract run(ctx: Context): Promise<void | unknown>;
}
```

## 📝 Creating Commands

### 1. Simple Command

```typescript
// src/modules/fun/commands/ping.ts
import { BaseCommand } from "../../../classes/command.js";
import type { Context } from "../../../classes/context.js";

export default class extends BaseCommand {
  description = "Check bot latency";

  async run(ctx: Context) {
    const start = Date.now();
    const msg = await ctx.reply("🏓 Pinging...");
    const end = Date.now();

    await msg.edit(`🏓 Pong! \`${end - start}ms\``);
  }
}
```

### 2. Command with Options

```typescript
// src/modules/utility/commands/say.ts
export default class extends BaseCommand {
  description = "Make the bot say something";
  usage = "say <message>";

  slashOptions = [
    {
      name: "message",
      description: "The message to say",
      type: ApplicationCommandOptionType.String,
      required: true
    }
  ];

  async run(ctx: Context) {
    const message = ctx.options.getString("message") || ctx.args.join(" ");

    if (!message) {
      return ctx.reply("Please provide a message!");
    }

    await ctx.reply(message);
  }
}
```

### 3. Advanced Command

```typescript
// src/modules/moderation/commands/ban.ts
export default class extends BaseCommand {
  description = "Ban a user from the server";
  usage = "ban <user> [reason]";
  examples = ["ban @user Spamming", "ban 123456789 Breaking rules"];

  userPerms = [PermissionFlagsBits.BanMembers];
  botPerms = [PermissionFlagsBits.BanMembers];

  slashOptions = [
    {
      name: "user",
      description: "User to ban",
      type: ApplicationCommandOptionType.User,
      required: true
    },
    {
      name: "reason",
      description: "Reason for ban",
      type: ApplicationCommandOptionType.String,
      required: false
    }
  ];

  async run(ctx: Context) {
    const user = ctx.options.getUser("user") || ctx.mentions.users.first();
    const reason = ctx.options.getString("reason") || ctx.args.slice(1).join(" ") || "No reason provided";

    if (!user) {
      return ctx.reply("Please mention a user to ban!");
    }

    try {
      await ctx.guild?.members.ban(user, { reason });
      await ctx.reply(`✅ Banned ${user.tag} for: ${reason}`);
    } catch (error) {
      await ctx.reply("❌ Failed to ban user. Check permissions.");
    }
  }
}
```

## ⚙️ Command Properties

### Basic Properties

```typescript
// Automatically set properties
name = "ping";           // From filename: ping.ts
category = "utility";    // From folder: modules/utility/

// Required property
description = "Check bot latency";

// Optional customization
usage = "ping";
aliases = ["p", "latency"];
examples = ["ping", "p"];
cooldown = 3000;         // 3 seconds
```

### Permission System

```typescript
// User permissions (what user needs)
userPerms = [
  PermissionFlagsBits.ManageMessages,
  PermissionFlagsBits.ManageChannels
];

// Bot permissions (what bot needs)
botPerms = [
  PermissionFlagsBits.SendMessages,
  PermissionFlagsBits.EmbedLinks
];

// Admin only (uses config.admins)
requireAdmin = true;

// NSFW channels only
nsfw = true;
```

### Slash Command Configuration

```typescript
// Enable/disable slash commands
slash = true;           // Creates slash command
slashOnly = false;      // Allow text commands too

// Slash command options
slashOptions = [
  {
    name: "target",
    description: "User to target",
    type: ApplicationCommandOptionType.User,
    required: true
  },
  {
    name: "amount",
    description: "Amount (1-100)",
    type: ApplicationCommandOptionType.Integer,
    required: false,
    min_value: 1,
    max_value: 100
  }
];
```

## 🎮 Context System

The `Context` class provides a unified interface for both text and slash commands:

### Basic Context Usage

```typescript
async run(ctx: Context) {
  // Send replies
  await ctx.reply("Hello!");
  await ctx.followUp("Additional message");

  // Access Discord objects
  const user = ctx.user;           // Command author
  const guild = ctx.guild;         // Server
  const channel = ctx.channel;     // Channel
  const member = ctx.member;       // Guild member

  // Get command arguments
  const args = ctx.args;           // Text command args
  const options = ctx.options;     // Slash command options
}
```

### Advanced Context Features

```typescript
async run(ctx: Context) {
  // Check command type
  if (ctx.isSlash) {
    // Slash command specific logic
    const user = ctx.options.getUser("user");
  } else {
    // Text command specific logic
    const user = ctx.mentions.users.first();
  }

  // Send different response types
  await ctx.reply({
    content: "Text message",
    embeds: [ctx.client.embed.setTitle("Title")],
    components: [ctx.client.row.addComponents(button)],
    ephemeral: true  // Only for slash commands
  });

  // Edit responses
  const msg = await ctx.reply("Loading...");
  await msg.edit("Done!");

  // Delete responses
  await ctx.delete();
}
```

## 🔄 Command Loading System

Commands are automatically loaded by the framework:

### Automatic Loading

```
modules/
├── utility/
│   └── commands/
│       ├── ping.ts      → Creates "ping" command
│       └── uptime.ts    → Creates "uptime" command
└── moderation/
    └── commands/
        └── ban.ts       → Creates "ban" command
```

### Loading Process

1. **Scan modules** - Framework scans `src/modules/` folders
2. **Find commands** - Looks for `commands/` subfolders
3. **Import files** - Dynamically imports each `.ts` file
4. **Set properties** - Auto-sets name and category
5. **Register slash** - Registers with Discord API
6. **Store commands** - Adds to `client.commands` collection

## 🎨 Command Patterns

### 1. Information Commands

```typescript
export default class extends BaseCommand {
  description = "Show server information";

  async run(ctx: Context) {
    const guild = ctx.guild!;

    const embed = ctx.client.embed
      .setTitle(guild.name)
      .setThumbnail(guild.iconURL())
      .addFields(
        { name: "Members", value: guild.memberCount.toString(), inline: true },
        { name: "Created", value: `<t:${Math.floor(guild.createdTimestamp / 1000)}:R>`, inline: true },
        { name: "Owner", value: `<@${guild.ownerId}>`, inline: true }
      );

    await ctx.reply({ embeds: [embed] });
  }
}
```

### 2. Interactive Commands

```typescript
export default class extends BaseCommand {
  description = "Play rock paper scissors";

  slashOptions = [
    {
      name: "choice",
      description: "Your choice",
      type: ApplicationCommandOptionType.String,
      required: true,
      choices: [
        { name: "Rock", value: "rock" },
        { name: "Paper", value: "paper" },
        { name: "Scissors", value: "scissors" }
      ]
    }
  ];

  async run(ctx: Context) {
    const userChoice = ctx.options.getString("choice") || ctx.args[0]?.toLowerCase();
    const botChoice = ["rock", "paper", "scissors"][Math.floor(Math.random() * 3)];

    let result: string;
    if (userChoice === botChoice) {
      result = "It's a tie!";
    } else if (
      (userChoice === "rock" && botChoice === "scissors") ||
      (userChoice === "paper" && botChoice === "rock") ||
      (userChoice === "scissors" && botChoice === "paper")
    ) {
      result = "You win!";
    } else {
      result = "You lose!";
    }

    await ctx.reply(`You: ${userChoice} | Bot: ${botChoice}\n${result}`);
  }
}
```

### 3. Database Commands

```typescript
export default class extends BaseCommand {
  description = "Check your balance";

  async run(ctx: Context) {
    const userId = ctx.user.id;
    const balance = ctx.client.db.get(`balance_${userId}`) || 0;

    const embed = ctx.client.embed
      .setTitle("💰 Balance")
      .setDescription(`You have **${balance}** coins!`)
      .setFooter({ text: `Requested by ${ctx.user.tag}`, iconURL: ctx.user.displayAvatarURL() });

    await ctx.reply({ embeds: [embed] });
  }
}
```

## 🚫 Error Handling

### Built-in Error Handling

```typescript
export default class extends BaseCommand {
  description = "Command that might fail";

  async run(ctx: Context) {
    try {
      // Risky operation
      const data = await someApiCall();
      await ctx.reply(`Success: ${data}`);
    } catch (error) {
      // Error automatically logged by framework
      await ctx.reply("❌ Something went wrong!");
    }
  }
}
```

### Custom Error Messages

```typescript
export default class extends BaseCommand {
  description = "Delete messages";

  async run(ctx: Context) {
    if (!ctx.member?.permissions.has(PermissionFlagsBits.ManageMessages)) {
      return ctx.reply("❌ You need Manage Messages permission!");
    }

    if (!ctx.guild?.members.me?.permissions.has(PermissionFlagsBits.ManageMessages)) {
      return ctx.reply("❌ I need Manage Messages permission!");
    }

    // Continue with command logic...
  }
}
```

## 🎯 Best Practices

### 1. Input Validation

```typescript
async run(ctx: Context) {
  const amount = parseInt(ctx.args[0]);

  if (isNaN(amount) || amount < 1 || amount > 100) {
    return ctx.reply("Please provide a number between 1 and 100!");
  }

  // Continue with valid input...
}
```

### 2. Permission Checks

```typescript
async run(ctx: Context) {
  // Check if command can be used in current context
  if (!ctx.guild) {
    return ctx.reply("This command can only be used in servers!");
  }

  if (ctx.channel?.type === ChannelType.DM) {
    return ctx.reply("This command doesn't work in DMs!");
  }
}
```

### 3. Consistent Responses

```typescript
// Use embeds for rich responses
const embed = ctx.client.embed
  .setTitle("✅ Success")
  .setDescription("Command executed successfully!")
  .setColor("#00FF00");

await ctx.reply({ embeds: [embed] });
```

---

**Next:** Learn about [Event Handling](./06-events.md) to respond to Discord events.

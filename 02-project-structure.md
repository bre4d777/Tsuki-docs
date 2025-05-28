# Project Structure

Understanding the project structure is key to working effectively with the Tsuki bot framework. This guide explains how everything is organized.

## 📂 Root Directory

```
tsuki-bot/
├── src/                    # TypeScript source code
├── app/                    # Compiled JavaScript (auto-generated)
├── node_modules/           # Dependencies
├── .env                    # Environment variables
├── .env.example            # Environment template
├── package.json            # Project configuration
├── tsconfig.json           # TypeScript configuration
├── eslint.config.js        # Linting rules
├── .prettierrc.json        # Code formatting rules
└── .gitignore              # Git ignore patterns
```

## 🎯 Source Code Structure (`src/`)

### Core Architecture

```
src/
├── index.ts                # Entry point & sharding manager
├── client.ts               # Main client initialization
├── config.ts               # Bot configuration
├── database.ts             # Database connection
└── typings/                # TypeScript definitions
    └── events.ts           # Event type definitions
```

### Classes (`src/classes/`)

Core classes that extend Discord.js functionality:

```
classes/
├── client.ts               # Extended Discord.js Client
├── command.ts              # Base command class
├── context.ts              # Command context wrapper
├── button.ts               # Extended button builder
└── paginator.ts            # Message pagination utility
```

### Events (`src/events/`)

Discord event handlers:

```
events/
├── ready.ts                # Bot startup event
├── interactionCreate.ts    # Slash commands & buttons
├── messageCreate.ts        # Text commands
├── messageUpdate.ts        # Message edit handling
├── guildCreate.ts          # Server join events
├── guildDelete.ts          # Server leave events
└── debug.ts                # Debug information
```

### Modules (`src/modules/`)

Feature-based modules:

```
modules/
├── admin/                  # Admin-only features
│   ├── commands/           # Admin commands
│   │   └── noPrefix.ts     # No-prefix command management
│   └── helpers/            # Admin utility functions
│       └── sendNoPrefixList.ts
└── example/                # Example implementations
    ├── commands/           # Example commands
    │   └── example.ts      # Basic command example
    └── events/             # Example events
        └── example.ts      # Basic event example
```

### Utilities (`src/utilities/`)

Helper functions and tools:

```
utilities/
├── loaders.ts              # Module & event loading
├── methods.ts              # General utility methods
├── chunk.ts                # Array chunking utility
├── processCommandContext.ts # Command processing
└── rateLimitManager.ts     # Rate limiting system
```

### Debugger (`src/debugger/`)

Advanced debugging tools:

```
debugger/
├── index.ts                # Main debugger class
└── commands/               # Debug commands
    ├── main.ts             # Debug command handler
    ├── js.ts               # JavaScript evaluation
    ├── sh.ts               # Shell command execution
    ├── curl.ts             # HTTP requests
    ├── cat.ts              # File reading
    └── rtt.ts              # Response time testing
```

## 🏗️ Architecture Patterns

### 1. Entry Point Flow

```
index.ts → ShardingManager → client.ts → ExtendedClient
```

1. **index.ts** - Creates sharding manager
2. **client.ts** - Initializes the extended client
3. **ExtendedClient** - Extends Discord.js with custom features

### 2. Module Loading

```
client.ts → loaders.ts → modules/ → commands/ & events/
```

1. Client starts up
2. Loaders scan module directories
3. Dynamically imports commands and events
4. Registers with Discord API

### 3. Command Processing

```
Message/Interaction → Event Handler → Context → Command.run()
```

1. Discord event received
2. Event handler processes input
3. Context object created
4. Command executed with context

## 📝 File Naming Conventions

### Commands
- **File name** = **Command name**
- Example: `ping.ts` creates command named "ping"
- Supports aliases through command properties

### Events
- **File name** = **Discord.js event name**
- Example: `messageCreate.ts` handles `messageCreate` event
- Must export default event object

### Modules
- **Folder name** = **Module category**
- Example: `admin/` creates "admin" category
- Affects command permissions and organization

## 🔧 Configuration Files

### TypeScript Configuration (`tsconfig.json`)
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "outDir": "./app",
    "strict": true,
    "moduleResolution": "node"
  }
}
```

### Package.json Scripts
```json
{
  "scripts": {
    "build": "npm run rm-app && tsc",
    "test": "node app/index.js",
    "lint": "eslint . --fix",
    "format": "prettier --write ."
  }
}
```

## 🎨 Code Organization Principles

### 1. Separation of Concerns
- **Classes** - Core functionality
- **Events** - Discord event handling
- **Modules** - Feature-specific code
- **Utilities** - Reusable helpers

### 2. Modular Design
- Each module is self-contained
- Commands and events grouped by feature
- Easy to add/remove functionality

### 3. Type Safety
- Full TypeScript implementation
- Strict type checking enabled
- Custom type definitions

### 4. Scalability
- Sharding support for large bots
- Modular loading system
- Efficient caching strategies

## 🚀 Development Workflow

### 1. Adding New Features
```
1. Create module folder in src/modules/
2. Add commands in module/commands/
3. Add events in module/events/
4. Framework auto-loads everything
```

### 2. Building & Testing
```bash
# Build TypeScript
npm run build

# Run bot
npm test

# Watch mode (development)
npx tsc --watch
```

### 3. Code Quality
```bash
# Format code
npm run format

# Lint code
npm run lint

# Check circular dependencies
npm run circular
```

## 📊 Database Structure

The bot uses SQLite with Quick.db wrapper:

```typescript
// Database location
database.ts → better-sqlite3 → quick.db wrapper

// Usage in code
client.db.get('key')
client.db.set('key', value)
client.db.delete('key')
```

## 🔍 Key Design Decisions

### Why This Structure?
1. **Modularity** - Easy to add/remove features
2. **Type Safety** - Catch errors at compile time
3. **Scalability** - Sharding and efficient caching
4. **Maintainability** - Clear separation of concerns
5. **Developer Experience** - Auto-loading and hot reloading

### Framework Benefits
- 🚀 **Fast Development** - Minimal boilerplate
- 🛡️ **Type Safety** - Full TypeScript support
- 📦 **Modular** - Plugin-like architecture
- 🔧 **Configurable** - Easy customization
- 🐛 **Debuggable** - Built-in debugging tools

---

**Next:** Learn about [Configuration](./03-configuration.md) to customize your bot settings.

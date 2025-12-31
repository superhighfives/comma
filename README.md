# Comma

A command-line command manager for saving, organizing, and executing frequently used shell commands.

## Features

- **Save commands** with memorable IDs (e.g., `comma dev` instead of typing `cd ~/Development && npm run dev`)
- **Placeholders** with optional defaults: `git push {remote:origin} {branch:main}`
- **Interactive TUI** for browsing, searching, and managing commands
- **Fuzzy search** across command IDs, text, and descriptions
- **Favorites** to prioritize your most-used commands
- **Usage tracking** with execution counts and timestamps
- **Shell integration** for commands that affect your current session (`cd`, `export`, etc.)
- **Import/Export** for backup and sharing command collections

## Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/comma.git
cd comma

# Install dependencies
bun install

# Link globally (optional)
bun link
```

### Shell Integration

For full functionality (especially commands like `cd` that need to run in your current shell), run:

```bash
comma setup
```

This adds a shell function to your config file (`.bashrc`, `.zshrc`, or `config.fish`) that enables direct command execution.

## Usage

### Interactive Mode

```bash
comma
```

Opens the TUI where you can browse, search, and execute commands.

**Keyboard Shortcuts:**

| Key | Action |
|-----|--------|
| `j` / `k` or `Up` / `Down` | Navigate list |
| `Enter` | Execute selected command |
| `n` | Add new command |
| `e` | Edit selected command |
| `d` | Delete selected command |
| `f` | Toggle favorite |
| `/` or `Ctrl+F` | Search |
| `Tab` | Toggle favorites-only filter |
| `q` or `Esc` | Quit |
| `Ctrl+R` | Refresh |

### CLI Commands

```bash
# Run a saved command
comma <id>

# Run with placeholder arguments
comma deploy staging   # Fills first placeholder with "staging"

# Add a new command
comma add <id> "<command>"
comma add dev "cd ~/projects && npm run dev"
comma add deploy "kubectl apply -f {file}"

# Edit a command
comma edit <id>                  # Opens TUI editor
comma edit <id> "<new-command>"  # Direct update

# Delete a command
comma delete <id>
comma rm <id>          # Alias

# List all commands
comma list
comma ls               # Alias
comma list -o json     # JSON output

# Search commands
comma search <query>

# Toggle favorite
comma fav <id>

# Show command details
comma info <id>

# Export/Import
comma export                     # Exports to stdout
comma export commands.json       # Exports to file
comma import commands.json       # Import from file

# Get raw command (for scripting)
comma -r <id>
```

### Placeholders

Commands can include placeholders that are filled at execution time:

```bash
# Required placeholder
comma add greet "echo Hello, {name}!"
comma greet World                    # → echo Hello, World!

# Optional placeholder with default
comma add push "git push {remote:origin} {branch:main}"
comma push                           # → git push origin main
comma push upstream                  # → git push upstream main
comma push upstream feature          # → git push upstream feature

# Escape braces when needed
comma add literal "echo \{not a placeholder\}"
```

## Configuration

Commands are stored in `~/.config/comma/commands.json`.

### Command Structure

```json
{
  "version": "1.0",
  "commands": [
    {
      "id": "dev",
      "command": "cd ~/projects && npm run dev",
      "description": "Start development server",
      "favorite": true,
      "createdAt": 1703980800000,
      "updatedAt": 1703980800000,
      "usageCount": 42,
      "lastUsed": 1704067200000
    }
  ]
}
```

## Tech Stack

- **[Bun](https://bun.sh)** - JavaScript runtime
- **[TypeScript](https://www.typescriptlang.org)** - Type safety
- **[React 19](https://react.dev)** - UI components
- **[OpenTUI](https://github.com/example/opentui)** - Terminal UI rendering
- **[Fuse.js](https://fusejs.io)** - Fuzzy search

## Project Structure

```
src/
  index.tsx           # CLI entry point
  cli/
    parser.ts         # Argument parsing
    formatters.ts     # Output formatting (table/JSON)
  core/
    placeholders.ts   # Placeholder parsing and substitution
    validation.ts     # ID and command validation
  storage/
    config.ts         # Config paths (~/.config/comma/)
    store.ts          # CRUD operations
    types.ts          # TypeScript interfaces
  tui/
    App.tsx           # Main TUI application
    CommandList.tsx   # Command list component
    CommandDialog.tsx # Add/Edit dialog
    DeleteDialog.tsx  # Delete confirmation
    PlaceholderDialog.tsx  # Placeholder input
    hooks.ts          # React hooks
  utils/
    fuzzy.ts          # Fuzzy search wrapper
    shell.ts          # Shell detection
scripts/
  build.ts            # Cross-platform build script
  release.ts          # npm publishing script
```

## Development

```bash
# Install dependencies
bun install

# Run in development mode (with hot reload)
bun dev

# Run directly
bun run src/index.tsx

# Type checking
bun run typecheck
```

## Building

Comma uses Bun's compile feature to create standalone executables for multiple platforms.

```bash
# Build for current platform
bun run build

# Build for all platforms
bun run build:all

# Build for specific target(s)
bun run scripts/build.ts --target darwin-arm64
bun run scripts/build.ts --target linux-x64,linux-arm64
```

**Supported targets:**

| Target | OS | Architecture |
|--------|-----|--------------|
| `darwin-arm64` | macOS | Apple Silicon |
| `darwin-x64` | macOS | Intel |
| `linux-x64` | Linux | x86_64 |
| `linux-arm64` | Linux | ARM64 |
| `win32-x64` | Windows | x86_64 |

Build output is written to `./dist/comma-{os}-{arch}/`.

## Releasing

The release script builds all platforms and publishes to npm.

```bash
# Dry run (test without publishing)
bun run release:dry

# Release to npm
NPM_TOKEN=your-token bun run release

# Release with a specific tag
NPM_TOKEN=your-token bun run release -- --tag beta

# Skip build (if already built)
NPM_TOKEN=your-token bun run release -- --skip-build
```

**Environment variables:**

| Variable | Description |
|----------|-------------|
| `NPM_TOKEN` | npm authentication token (required for publishing) |
| `NPM_TAG` | Distribution tag (default: `latest`) |
| `OUTPUT_DIR` | Build output directory (default: `./dist`) |
| `TARGETS` | Comma-separated targets or `all` |

## Validation Rules

- **Command ID**: Lowercase alphanumeric and hyphens only, max 50 characters
- **Command**: Max 2000 characters
- **Reserved IDs**: `add`, `edit`, `delete`, `rm`, `list`, `ls`, `search`, `fav`, `info`, `setup`, `init`, `export`, `import`, `help`

## Why Shell Integration?

Some commands (like `cd`, `export`, `alias`) must run in your current shell to have any effect. Without shell integration, comma runs commands in a subprocess, which means:

- `cd` changes directory in the subprocess, not your terminal
- `export` sets variables that disappear when the subprocess exits

The `comma setup` command installs a shell function that intercepts command execution and runs commands directly in your shell using `eval`.

## License

MIT

# ircord

Command-line management tool for IRCord server.

## Overview

`ircord` is a CLI binary for managing a running IRCord server instance. It connects to the server's admin socket (named pipe on Windows, Unix domain socket on Linux) and sends commands using the existing JSON-over-length-prefixed-frame protocol.

## Features

- **Builtin commands**: shutdown, restart, status, kick, ban, users, logtail, tui
- **Script commands**: extend with JavaScript (QuickJS) scripts in `~/.ircord/commands/`
- **Streaming**: `logtail` streams server logs in real-time
- **Interactive**: `tui` opens the full admin TUI

## Usage

```bash
# Server management
ircord status              # Show server uptime, version, user count
ircord shutdown            # Graceful server shutdown
ircord restart             # Restart the server
ircord update              # Pull latest + rebuild + restart

# User management
ircord users               # List online users
ircord kick <user> [reason]
ircord ban <user> [reason]

# Monitoring
ircord logtail             # Stream server logs (Ctrl+C to stop)
ircord logtail --level warn # Filter by log level

# Admin TUI
ircord tui                 # Open interactive admin TUI

# Help
ircord help                # List all commands
ircord help <command>      # Help for specific command
ircord version             # Show CLI version
```

## Script Commands

Place `.js` files in `~/.ircord/commands/` to add custom commands:

```js
// ~/.ircord/commands/deploy.js
export const command = {
    name: "deploy",
    description: "Pull latest and rebuild server",
};

export async function execute(ctx) {
    ctx.print("Pulling latest...");
    await ircord.exec("git", ["pull"]);
    ctx.print("Building...");
    await ircord.exec("cmake", ["--build", "build"]);
    await ctx.admin.send("restart", {});
    ctx.print("Done!");
}
```

Scripts have access to:
- `ctx.admin` — admin socket client (send commands, receive events)
- `ctx.print()` — output to terminal
- `ctx.args` — command-line arguments
- `ircord.exec()` — run shell commands
- `ircord.fs` — filesystem operations

## Building

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

## Architecture

```
ircord (CLI binary)
  |
  +-- CommandRegistry
  |     +-- BuiltinCommand (C++ compiled)
  |     +-- ScriptCommand  (QuickJS runtime)
  |
  +-- AdminSocketClient (from ircord-server-tui library)
  |     +-- Named pipe (Windows) / Unix socket (Linux)
  |     +-- 4-byte BE length prefix + JSON
  |
  +-- ScriptEngine (QuickJS, from ircord-plugin)
        +-- ircord.exec, ircord.fs APIs
        +-- ctx.admin bridge to AdminSocketClient
```

## Dependencies

- [ircord-server-tui](https://github.com/hittoSepi/ircord-server-tui) — AdminSocketClient, admin protocol
- [ircord-plugin](../ircord-plugin/) — QuickJS engine for script commands
- [nlohmann/json](https://github.com/nlohmann/json) — JSON serialization
- [Boost.Asio](https://www.boost.org/) — async I/O for admin socket

## License

Part of the IRCord project.

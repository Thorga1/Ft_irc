# ft_irc

An IRC server written in C++98, built as part of the **42 school** curriculum. `ircserv` handles multiple clients at once on a single thread using non-blocking sockets and `poll()`, and implements the core of the IRC protocol: authentication, nicknames, channels, private messages and channel operator commands.

It works with real IRC clients like irssi, WeeChat or HexChat, as well as plain `nc`.

---

## Features

**Server**
- Single `poll()` loop handling every connection, with non-blocking sockets and no forking or threads
- Password-protected connections
- Message buffering, so commands split across several packets are put back together before being processed
- Clean shutdown on `SIGINT`, `SIGTERM` or EOF on standard input
- Port validation (`1-65535`)

**Users**
- Registration with `PASS`, `NICK` and `USER`
- Unique nicknames, which can be changed at any time
- Private messages between users

**Channels**
- Create and join channels starting with `#` or `&`
- Messages broadcast to every member of a channel
- Channel operators with special privileges (the creator becomes operator)
- Channel topics

---

## Supported commands

| Command | Usage | Description |
|---------|-------|-------------|
| `PASS`    | `PASS <password>` | Authenticate with the server password (must come first) |
| `NICK`    | `NICK <nickname>` | Set or change your nickname |
| `USER`    | `USER <username> 0 * :<realname>` | Complete registration |
| `JOIN`    | `JOIN <#channel> [key]` | Join or create a channel |
| `PRIVMSG` | `PRIVMSG <target> :<message>` | Message a user or channel (`MSG` also works) |
| `TOPIC`   | `TOPIC <#channel> [:topic]` | View or change a channel's topic |
| `KICK`    | `KICK <#channel> <nick> [:reason]` | Remove a user from a channel (operators only) |
| `INVITE`  | `INVITE <nick> <#channel>` | Invite a user to a channel |
| `MODE`    | `MODE <#channel> <+/-flags> [args]` | Change channel modes (operators only) |
| `WHOIS`   | `WHOIS <nick>` | Get information about a user |
| `PING`    | `PING <token>` | Keep the connection alive |
| `QUIT`    | `QUIT [:message]` | Disconnect from the server |

### Channel modes

| Mode | Description |
|------|-------------|
| `i` | Invite-only channel |
| `t` | Only operators can change the topic |
| `k` | Channel key (password) required to join |
| `o` | Give or take operator privileges |
| `l` | Limit the number of users in the channel |

---

## Installation

### Requirements
- A Unix-like system (Linux or macOS)
- A C++ compiler with C++98 support (`c++`, `g++` or `clang++`)
- `make`

### Build
```bash
git clone https://github.com/Thorga1/ft_irc.git
cd ft_irc
make
```

### Run
```bash
./ircserv <port> <password>
```

Example:
```bash
./ircserv 6667 mypassword
```

### Make targets
| Target | Description |
|--------|-------------|
| `make` / `make all` | Builds `ircserv` |
| `make clean` | Removes object files |
| `make fclean` | Removes object files and the binary |
| `make re` | Full rebuild |

---

## Connecting

### With irssi
```
/connect localhost 6667 mypassword yournick
/join #general
```

### With netcat
```bash
nc -C localhost 6667
```
Then register by hand:
```
PASS mypassword
NICK bob
USER bob 0 * :Bob
JOIN #general
PRIVMSG #general :hello everyone
```

`-C` makes `nc` send `\r\n` line endings, like a real IRC client. To test partial messages, type part of a command and press `Ctrl-D` to send it without a newline, then finish it. The server waits until the full line arrives.

---

## Project structure

```
ft_irc/
├── includes/
│   ├── server.hpp           # Server: sockets, poll loop, clients, channels
│   ├── client.hpp           # Client state and input buffer
│   ├── channel.hpp          # Channel members, operators, modes
│   ├── Client_handler.hpp   # Parses lines and dispatches commands
│   ├── ACommand.hpp         # Abstract base class for commands
│   └── <Command>.hpp        # One class per IRC command
├── src/
│   ├── main.cpp             # Argument checks, signal setup, server start
│   ├── server.cpp
│   ├── client.cpp
│   ├── channel.cpp
│   ├── Client_handler.cpp
│   └── commands/            # Join, Kick, Mode, Nick, Pass, Privmsg, ...
└── Makefile
```

### How a message is handled

1. **Poll**: the server waits on `poll()` for activity on the listening socket, client sockets and standard input.
2. **Accept**: a new connection is accepted, set to non-blocking, and added to the poll list.
3. **Read**: incoming data is appended to that client's buffer.
4. **Split**: each complete line is pulled out of the buffer; incomplete data stays there until the rest arrives.
5. **Dispatch**: the command name is looked up in a map of command objects, each inheriting from `ACommand`. Clients must register with `PASS` before anything else.
6. **Reply**: the command runs and sends numeric replies or broadcasts to the relevant users and channels.

---

## Authors

- **Thorgal Ordner** ([@Thorga1](https://github.com/Thorga1)), `tordner` @ 42
- **Leo Garcia**, `leogarci` @ 42
- **Amazigh Lasri**, `amlasri` @ 42


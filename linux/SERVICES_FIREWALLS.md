# Linux Mint Dev Setup
A collection of notes and scripts for setting up my development environment on Linux Mint.

## 🛠 Installed Tools
*   **GCC (build-essential)**: C/C++ compiler and build tools.
*   **Python 3**: For scripting and automation.
*   **Apache2**: Local web server.
*   **OpenSSH Server**: For remote access (Service name: `ssh`).

## 🚀 Getting Started
### Updating the System
Always run these first on a new Mint install:
```bash
sudo apt update && sudo apt upgrade -y
```

### Running the Hello World Program (C)
1. Compile: `gcc hello.c -o hello`
2. Run: `./hello`

### Running the Python Script
Run directly using the interpreter:
```bash
python3 hello.py
```

## 📋 Useful Commands
*   `sudo ufw status`: Check firewall status.
*   `systemctl status ssh`: Check if SSH server is running.
*   `ip addr`: Find your local IP address.

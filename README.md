# wlan-deauth-tool

Automated WiFi deauthentication tool that targets specific MAC addresses from a blacklist file. Designed to run on Linux systems with external USB WiFi adapters (e.g. Raspberry Pi).

## Features

- **Parallel workers** — Launches one dedicated worker per target MAC, all attacking simultaneously instead of sequentially.
- **Throttled startup** — Workers are staggered in batches (`MAX_PARALLEL` + `STAGGER_DELAY`) to avoid saturating the adapter on launch.
- **Dynamic interface resolution** — Identifies adapters by their kernel driver name instead of `wlanX`, which can change across reboots or reconnections.
- **Dual-band support** — Separate configurations for 5GHz and 2.4GHz attacks, each with its own driver, channel, and AP BSSID.
- **Channel watchdog** — Background process that periodically checks the interface is still on the correct channel and resets it if it drifts.
- **MAC spoofing** — Applies a custom MAC address before enabling monitor mode.
- **Fault tolerance** — Individual worker failures do not stop the attack; each worker retries automatically.
- **Clean shutdown** — Traps `SIGINT`/`SIGTERM` to stop all workers and the watchdog before exiting.
- **External config** — All parameters are loaded from a config file, keeping sensitive data out of the script.

## Requirements

- Linux with wireless extensions (`iw`, `ip`)
- `aireplay-ng` (part of the [aircrack-ng](https://www.aircrack-ng.org/) suite)
- `macchanger`
- One or more external USB WiFi adapters that support monitor mode

## Setup

1. Clone the repository:

```bash
git clone https://github.com/YOUR_USER/wlan-deauth-tool.git
cd wlan-deauth-tool
```

2. Copy the example config and fill in your values:

```bash
cp config.example config
nano config
```

3. Create a blacklist file with one target MAC address per line:

```bash
echo "AA:BB:CC:DD:EE:FF" >> ~/blacklist_home_wlan
```

4. Make sure the script is executable:

```bash
chmod +x deauth-blacklist
```

## Usage

```bash
# 5GHz attack
./deauth-blacklist 5g

# 2.4GHz attack
./deauth-blacklist 24g
```

You can override the config file path with the `CONFIG_FILE` environment variable:

```bash
CONFIG_FILE=/path/to/my/config ./deauth-blacklist 5g
```

Run both bands simultaneously in separate terminals for full coverage:

```bash
# Terminal 1
./deauth-blacklist 5g

# Terminal 2
./deauth-blacklist 24g
```

Stop the attack with `Ctrl+C`. All workers and the watchdog are terminated automatically.

## Configuration

See [config.example](config.example) for all available options:

| Variable | Description |
|---|---|
| `DRIVER_5G` | Kernel driver name for the 5GHz adapter |
| `DRIVER_24G` | Kernel driver name for the 2.4GHz adapter |
| `CHANNEL_5G` | Channel to lock the 5GHz interface to |
| `CHANNEL_24G` | Channel to lock the 2.4GHz interface to |
| `AP_BSSID_5G` | Target access point BSSID on 5GHz |
| `AP_BSSID_24G` | Target access point BSSID on 2.4GHz |
| `MAC_SPOOF` | MAC address to spoof before enabling monitor mode |
| `BLACKLIST` | Path to the file with target MAC addresses |
| `WATCHDOG_INTERVAL` | Seconds between channel drift checks (default: 30) |
| `DEAUTH_COUNT` | Deauth bursts per cycle per worker (default: 1) |
| `DEAUTH_SLEEP` | Seconds between bursts within each worker (default: 0.5) |
| `MAX_PARALLEL` | Workers launched per batch before staggering (default: 4) |
| `STAGGER_DELAY` | Seconds between batches at startup (default: 0.3) |

To find your adapter's driver name:

```bash
basename "$(readlink /sys/class/net/wlan0/device/driver)"
```

## Tuning

If the adapter becomes unstable under load, adjust these values in `config`:

- **Reduce concurrency**: lower `MAX_PARALLEL` (e.g. `3`) or raise `STAGGER_DELAY` (e.g. `0.5`)
- **Slower attack rate**: raise `DEAUTH_SLEEP` (e.g. `1`)
- **More aggressive**: lower `DEAUTH_SLEEP` (e.g. `0.2`) — use with caution

## Disclaimer

This tool is intended for authorized network auditing and security testing only. Unauthorized use against networks you do not own or have explicit permission to test is illegal. Use responsibly.

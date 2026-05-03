# minimal-tunnel

**Connect your local database to [Minimal](https://littlebit.in) — no open ports, no firewall rules, no VPN.**

`minimal-tunnel` is a lightweight program you run on the machine where your database lives. It opens a single outbound encrypted connection to Minimal test infrastructure, letting you experience the full Minimal feature set against a database that is running locally on your system. Setup takes under a minute.

---

## How it works

The agent dials out over TLS on port 443 — the same port used by HTTPS. Your database **never** needs to be exposed to the internet, to experience Minimal. Minimal routes queries through the tunnel in both directions.

```
Your machine                        Minimal
┌──────────────────────┐           ┌──────────────────────┐
│  Database  :5432     │           │                      │
│       │              │           │  Minimal UI / API    │
│       ▼              │           │         │            │
│  minimal-tunnel  ────┼── TLS ───►│  Tunnel Endpoint     │
│                      │  :443     │                      │
└──────────────────────┘           └──────────────────────┘
```

- **Outbound-only.** The agent only dials out. Nothing on your machine listens for external traffic.
- **Port-scoped.** Traffic is forwarded exclusively to the local port you specify — nothing else on your machine is reachable through the tunnel.
- **Session-bound.** Each tunnel is tied to a short-lived code generated in the [Minimal UI](https://littlebit.in/try) . The code expires in minutes if unused.
- **Stops cleanly.** Press Ctrl+C at any time. The tunnel closes immediately and the session is invalidated.

---

## Security

| Property | What it means for you |
|---|---|
| **Outbound-only** | No listening ports are opened on your machine |
| **TLS encrypted** | All traffic between your machine and Minimal is encrypted in transit |
| **Port-scoped** | Only the port you pass to `--local-port` is reachable — nothing else |
| **No credential access** | The agent proxies raw TCP; it never reads or stores your database password |
| **No persistence** | The agent writes nothing to disk |
| **No root required** | Standard user privileges are sufficient on all platforms |
| **Short-lived sessions** | Codes expire quickly; each connection flow generates a new one |

The agent does not inspect database traffic. Your database's own authentication — passwords, TLS, certificates — applies end-to-end without modification.

---

## Download

| Platform | Download |
|---|---|
| macOS — Apple Silicon | [minimal-tunnel-darwin-arm64](https://github.com/littlebitlabs/Minmux/blob/main/dist/minimal-tunnel-darwin-amd64) |
| macOS — Intel | [minimal-tunnel-darwin-amd64](https://github.com/littlebitlabs/Minmux/blob/main/dist/minimal-tunnel-darwin-amd64) |
| Linux — x86-64 | [minimal-tunnel-linux-amd64](https://github.com/littlebitlabs/Minmux/blob/main/dist/minimal-tunnel-linux-amd64) |
| Linux — ARM64 | [minimal-tunnel-linux-arm64](https://github.com/littlebitlabs/Minmux/blob/main/distminimal-tunnel-linux-arm64) |
| Windows — x86-64 | [minimal-tunnel-windows-amd64.exe](https://github.com/littlebitlabs/Minmux/blob/main/dist/minimal-tunnel-windows-amd64.exe) |

Make the binary executable on macOS and Linux before running it:

```bash
chmod +x ./minimal-tunnel-*
```

---

## macOS — first-run setup

macOS blocks binaries downloaded from the internet that are not signed by a registered Apple developer. On first run you will see:

> *"minimal-tunnel-darwin-arm64" cannot be opened because Apple cannot check it for malicious software.*

This is a Gatekeeper restriction on unsigned binaries, not an indication that the software is harmful. Use either method below to allow it. We are working on signing and notarising future releases to remove this step entirely.

### Option A — Terminal (recommended)

You are already in a terminal to run the agent. Run this once, using the exact filename you downloaded:

```bash
xattr -d com.apple.quarantine ./minimal-tunnel-darwin-arm64
```

No further steps needed. Run the agent as normal from this point on.

### Option B — System Settings

1. Attempt to run the binary. macOS will block it and show the dialog above — click **OK** to dismiss it.
2. Open **System Settings → Privacy & Security**.
3. Scroll to the Security section. You will see:  
   *"minimal-tunnel-darwin-arm64" was blocked from use because it is not from an identified developer.*
4. Click **Open Anyway**, then confirm with your macOS password.

---

## Usage

```
./minimal-tunnel --code <CODE> --local-port <PORT> --server <SERVER>
```

| Flag | Required | Description |
|---|---|---|
| `--code` | Yes | Session code shown in the Minimal UI |
| `--local-port` | Yes | Port your database is listening on locally |
| `--server` | Yes | Tunnel server address provided by Minimal |


### Examples

**PostgreSQL on the default port:**

```bash
./minimal-tunnel-darwin-arm64 \
  --code        ABC123 \
  --local-port  5432 \
  --server      https://play.autoapi.littlebit.in
```

**MySQL on a non-standard port:**

```bash
./minimal-tunnel-darwin-arm64 \
  --code        XYZ789 \
  --local-port  3807 \
  --server      https://play.autoapi.littlebit.in
```

**ClickHouse on a non-standard port:**

```bash
./minimal-tunnel-darwin-arm64 \
  --code        XYZ789 \
  --local-port  9900 \
  --server      https://play.autoapi.littlebit.in
```

**MariaDB on a non-standard port:**

```bash
./minimal-tunnel-darwin-arm64 \
  --code        XYZ789 \
  --local-port  3407 \
  --server      https://play.autoapi.littlebit.in
```

---

## Expected output

Once the tunnel is established you will see:

```
minimal-tunnel v1.0.0
Connecting to tunnel.minimal.in ...
Tunnel established — forwarding server port 24510 to local port 5432
```

Return to the Minimal UI and click **I've started the tunnel**. Minimal will verify the connection and proceed automatically.

To stop the tunnel, press **Ctrl+C**:

```
^C
Shutting down.
```

---

## Troubleshooting

**`Error: tunnel code not found — get a new code from the Minimal UI`**

The code has expired or was already used. Return to the Minimal UI and start a new connection flow to get a fresh code.

**`Connection lost (...). Retrying in 2s (attempt 1) ...`**

The agent lost contact with the tunnel endpoint and is retrying automatically. This typically resolves on its own within a few seconds. If it persists, verify that outbound traffic on port 443 is not blocked on your network or by a corporate proxy.

**`Warning: could not reach local database at 127.0.0.1:5432`**

The tunnel to Minimal connected successfully, but the agent could not reach your local database on the specified port. Verify your database is running and listening on the port you passed to `--local-port`.

---

## Compatibility

`minimal-tunnel` forwards raw TCP. It works with any database or service that communicates over TCP:

- PostgreSQL
- ClickHouse
- MySQL
- MariaDB


No driver changes, no ORM configuration, and no client-side modifications are required. Connect to the database exactly as you normally would — the tunnel is transparent to both sides.

---

## Disclaimer

`minimal-tunnel` is provided as-is, without warranty of any kind, express or implied. By downloading and running this software, you accept full responsibility for its use on your systems and infrastructure. Littlebit Labs bears no liability for data loss, data corruption, unauthorised access, security incidents, or any other damages — direct, indirect, or consequential — arising from your use of this software. You are solely responsible for securing your database credentials, your local network configuration, and the environment in which this agent runs. Continued use of this software constitutes acceptance of these terms.

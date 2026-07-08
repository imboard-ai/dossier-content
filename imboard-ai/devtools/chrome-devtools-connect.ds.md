---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "chrome-devtools-connect",
  "title": "Chrome DevTools MCP — Connect & Self-Heal the WSL→Windows Bridge",
  "version": "1.0.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "last_updated": "2026-07-08",
  "objective": "Get the chrome-devtools MCP driving a real Windows Chrome from a WSL2 session — launch the browser if needed, diagnose and repair the debug-port bridge (Chrome 149+ binds ::1 only; a stale portproxy strands it), then verify end-to-end control with navigate + screenshot.",
  "category": [
    "devops"
  ],
  "tags": [
    "chrome-devtools",
    "mcp",
    "wsl2",
    "browser-automation",
    "portproxy",
    "live-verify",
    "setup"
  ],
  "risk_level": "medium",
  "requires_approval": false,
  "content_scope": "references-external",
  "external_references": [
    {
      "url": "http://172.18.208.1:9222",
      "description": "Chrome DevTools Protocol debug endpoint on the WSL-facing Windows host IP — the fixed browserUrl the chrome-devtools MCP attaches to. The IP is the local WSL vEthernet gateway, not a public host.",
      "type": "api",
      "trust_level": "trusted",
      "required": true
    },
    {
      "url": "http://172.18.208.1:9222/json/version",
      "description": "CDP version endpoint; returns Chrome-version JSON when the debug bridge is up. Used as the reachability probe.",
      "type": "api",
      "trust_level": "trusted",
      "required": true
    }
  ],
  "risk_factors": [
    "network_access",
    "system_configuration",
    "executes_external_code"
  ],
  "destructive_operations": [
    "Replaces the Windows netsh portproxy rule on port 9222 (v4tov4 → v4tov6) — requires one-time UAC elevation",
    "Adds a Windows Defender Firewall inbound allow rule for TCP 9222",
    "Launches a Windows Chrome process against an isolated debug profile"
  ],
  "estimated_duration": {
    "min_minutes": 1,
    "max_minutes": 4
  },
  "tools_required": [
    {
      "name": "curl",
      "check_command": "curl --version"
    },
    {
      "name": "powershell.exe",
      "check_command": "powershell.exe -NoProfile -Command \"$PSVersionTable.PSVersion.ToString()\""
    },
    {
      "name": "chrome-devtools MCP",
      "check_command": "curl -s --max-time 5 http://172.18.208.1:9222/json/version"
    }
  ],
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "332be9aa7a58eca011cf6c00fe19155b103218081b20fe81d6ec9e541cb6ae75"
  }
}
---

# Chrome DevTools MCP — Connect & Self-Heal the WSL→Windows Bridge

## Objective

Reach a state where the `chrome-devtools` MCP tools (`list_pages`, `navigate_page`,
`take_screenshot`, `click`, `fill`, …) actually drive a **real, user-visible Windows
Chrome** from inside a **WSL2** Claude Code session — so you can do live UI verification
and authenticated (OTP / Google SSO) flows the user can watch and complete.

The MCP does **not** launch a browser. It is configured with a fixed
`--browserUrl=http://172.18.208.1:9222` (the Windows host as seen from WSL2). Chrome must
already be listening there. On Chrome **149+** this is non-trivial: Chrome force-binds the
debug port to loopback **`::1` (IPv6) only** and ignores `--remote-debugging-address=0.0.0.0`,
so a Windows `netsh portproxy` bridge is required — and a stale IPv4 (`v4tov4`) bridge
actively strands the connection. This dossier detects that and repairs it.

Optimize for the common case: run it, and within a minute either you're already connected,
or Chrome gets launched and the bridge is auto-repaired (one UAC click) and you're connected.

## Environment assumptions

- Host is **WSL2** on Windows; Windows Chrome is installed at
  `C:\Program Files\Google\Chrome\Application\chrome.exe`.
- The `chrome-devtools` MCP is registered in this session with
  `--browserUrl=http://172.18.208.1:9222`.
- `172.18.208.1` is the Windows-side vEthernet (WSL) IP. **It can change** across reboots /
  WSL restarts. If it changes, the MCP's own config must be updated to match — that is out of
  scope here (this dossier assumes the MCP points where Chrome's bridge listens). The portproxy
  we install listens on `0.0.0.0`, so it serves whatever that Windows IP currently is.

## Constraints

### Hard rules

- **Never `taskkill /IM chrome.exe`.** That would kill the user's normal browser. The debug
  instance runs an **isolated `--user-data-dir`**; if you must restart it, target it by that
  profile/PID, never by image name.
- **The user completes all authentication.** Passwordless OTP, Google SSO, and 2FA are the
  user's to enter. Never attempt to type a password or a 2FA code. Drive up to the auth wall,
  screenshot it, hand back, and resume after the user says they're in.
- **One-time elevation only.** The portproxy + firewall repair needs admin **once**; the
  netsh portproxy rule is persistent across reboots. Do not re-elevate on every run — only when
  the reachability check actually fails.
- **Leave Chrome running for the user to close.** The Windows Chrome is a WSL-interop process;
  `pkill -f` from WSL won't reach it. Do not try to tear it down.

### Degraded modes

- **No admin / UAC declined** — you cannot fix the portproxy. Fall back to driving the local app
  headlessly with Playwright/Chromium *inside WSL* for autonomous verification only (the user
  can't see or log into that instance). State this limitation explicitly; do not claim live
  verification you can't show.
- **App/site needs login and the user is away** — navigate, screenshot the auth screen, report
  that login is required, and stop.

## Workflow

### 1. Fast path — is it already connected?

```bash
curl -s --max-time 5 http://172.18.208.1:9222/json/version
```

- Returns Chrome-version JSON → the bridge is up. **Skip to step 5** (verify via MCP).
- Empty / connection error → continue.

### 2. Launch Windows Chrome with remote debugging

Isolated profile so it never touches the user's real browser or cookies:

```bash
"/mnt/c/Program Files/Google/Chrome/Application/chrome.exe" \
  --remote-debugging-port=9222 --remote-debugging-address=0.0.0.0 \
  --user-data-dir="C:\\temp\\chrome-debug-imboard" \
  about:blank >/dev/null 2>&1 &
```

Then poll for readiness (Chrome takes a couple seconds):

```bash
for i in $(seq 1 8); do
  out=$(curl -s --max-time 3 http://172.18.208.1:9222/json/version)
  [ -n "$out" ] && { echo "REACHABLE"; break; }
  sleep 1
done
```

- Reachable → **skip to step 5**.
- Still empty after ~8s → the bridge is broken; continue to diagnose.

### 3. Diagnose the bridge

```bash
# What is actually listening on Windows :9222, and how is the bridge wired?
powershell.exe -NoProfile -Command "Get-NetTCPConnection -LocalPort 9222 -State Listen | Select LocalAddress,LocalPort,State | Format-Table -Auto"
netsh.exe interface portproxy show all
```

**Signature of the Chrome-149 failure** (the common one):

- A listener exists on **`::1:9222`** (that's Chrome) and on **`0.0.0.0:9222`** (that's the
  portproxy), but **nothing on IPv4 `127.0.0.1:9222`**.
- The portproxy rule reads `0.0.0.0:9222 → 127.0.0.1:9222` (a **v4tov4** rule pointing at a
  **dead** target).

Root cause: the persistent `v4tov4` portproxy grabbed IPv4 `0.0.0.0:9222` first, so Chrome
couldn't bind IPv4 `127.0.0.1` and fell back to `::1` only. WSL's request to
`172.18.208.1:9222` hits the portproxy, which forwards to `127.0.0.1:9222` where nothing
listens → the connection dies.

Confirm which loopback Chrome answers on (note the `${addr}` braces — writing `$addr` followed by
a colon makes PowerShell parse it as a scoped variable and silently emit a malformed URL with an
empty host):

```bash
powershell.exe -NoProfile -Command '
foreach ($addr in @("127.0.0.1","[::1]")) {
  try { $r = Invoke-WebRequest "http://${addr}:9222/json/version" -UseBasicParsing -TimeoutSec 3
        Write-Output "$addr -> OK ($($r.StatusCode))" }
  catch { Write-Output "$addr -> FAIL" }
}'
```

Expected on the broken state: `[::1] -> OK`, `127.0.0.1 -> FAIL`.

### 4. Repair the bridge (one-time UAC)

Repoint the portproxy from IPv4 to **IPv6 loopback (`::1`)** so it forwards to where Chrome
actually listens, and open the firewall for the WSL subnet. `netsh portproxy` and
`New-NetFirewallRule` both require admin, so trigger an elevated PowerShell (the user clicks
**Yes** on the UAC dialog):

```bash
# Write the fix script to a Windows-visible path
cat > /mnt/c/temp/fix-chrome-bridge.ps1 <<'PS'
netsh interface portproxy delete v4tov4 listenport=9222 listenaddress=0.0.0.0 2>$null
netsh interface portproxy delete v4tov6 listenport=9222 listenaddress=0.0.0.0 2>$null
netsh interface portproxy add    v4tov6 listenport=9222 listenaddress=0.0.0.0 connectport=9222 connectaddress=::1
if (-not (Get-NetFirewallRule -DisplayName "WSL Chrome Debug 9222" -ErrorAction SilentlyContinue)) {
  New-NetFirewallRule -DisplayName "WSL Chrome Debug 9222" -Direction Inbound -LocalPort 9222 -Protocol TCP -Action Allow | Out-Null
}
"--- portproxy now ---"; netsh interface portproxy show all
PS

# Tell the user a UAC dialog is coming and to click YES, then elevate
echo ">>> A Windows UAC dialog will appear — click YES. <<<"
powershell.exe -NoProfile -Command "Start-Process powershell -Verb RunAs -Wait -ArgumentList '-NoProfile','-ExecutionPolicy','Bypass','-File','C:\temp\fix-chrome-bridge.ps1'"
```

If the user declines UAC you'll see `The operation was canceled by the user` — ask them to
approve and retry the elevation once; do not loop on it.

Re-verify from WSL:

```bash
curl -s --max-time 5 http://172.18.208.1:9222/json/version
```

Now the rule should read `Listen 0.0.0.0:9222 → Connect ::1:9222` and the curl returns JSON.

### 5. Verify end-to-end control via the MCP

Do not declare success on the curl alone — prove the MCP itself drives Chrome:

1. `list_pages` — should list the open tab(s).
2. `navigate_page` to a self-contained page first (no DNS dependency):
   `data:text/html,<h1>chrome-devtools MCP OK</h1>`
3. `take_screenshot` — confirm the rendered page comes back.

Only after a screenshot round-trips is the connection proven. (A `navigate` to an external URL
may fail with `ERR_NAME_NOT_RESOLVED` on a cold debug profile — that's transient profile DNS,
not the MCP; retry or reload. Google/most domains resolve fine once warm.)

### 6. Authenticated flows (hand-off pattern)

The isolated debug profile starts logged out. To drive an authenticated app:

- **imboard (passwordless email OTP):** fill email → "Send sign-in code". In dev the backend
  returns the code in the `POST /api/auth/signuplogin` response (`{"loginCode":"NNNNNN"}`,
  gated by `shouldExposeDevAuthCode`) and stores it on `user.loginCode`; grep the dev-server
  log. Dev user `a@gmail.com` has the imported **"(Example) Stellar Dynamics Inc."** board.
- **Google / third-party SSO:** navigate, screenshot the account-chooser / password / 2FA
  screen, and **stop** — the user signs in, then says "I logged in". Re-`list_pages` to confirm
  the destination loaded, then resume driving. The profile caches the session, so subsequent
  runs land straight on the app.

## Known Pitfalls

- **Chrome 149+ ignores `--remote-debugging-address=0.0.0.0`** and binds `::1` only. The address
  flag is not a fix; the portproxy is.
- **A stale `v4tov4` portproxy is worse than none** — it occupies IPv4 `0.0.0.0:9222`, blocking
  Chrome's IPv4 bind *and* forwarding to a dead target. The repair deletes it before adding the
  `v4tov6` rule.
- **PowerShell `$addr:9222` gotcha** — the colon makes PowerShell read `$addr:` as a drive/scope
  qualifier, yielding a scheme with an empty host (`http://` + nothing). Always use `${addr}`.
- **`taskkill.exe /IM chrome.exe` nukes the user's real Chrome.** The debug instance is isolated
  by `--user-data-dir`; target it by profile/PID if a restart is ever needed.
- **WSL host IP drift** — `172.18.208.1` can change; the portproxy listens on `0.0.0.0` so it
  keeps serving, but the MCP's fixed `--browserUrl` would need updating if the IP moves (out of
  scope here).
- **Second `chrome.exe` launch with the same `--user-data-dir`** just opens a tab in the existing
  instance and does **not** re-bind the port — don't expect a relaunch to change the binding.
- **Don't over-trust `curl`.** Reachability ≠ control. Always finish with an MCP `take_screenshot`.

## Validation

- [ ] `curl http://172.18.208.1:9222/json/version` returns Chrome-version JSON.
- [ ] `netsh interface portproxy show all` shows `0.0.0.0:9222 → ::1:9222` (a `v4tov6` rule).
- [ ] MCP `list_pages` returns the open tab(s).
- [ ] MCP `navigate_page` + `take_screenshot` round-trips a rendered page.
- [ ] No `chrome.exe` was killed by image name; the user's normal browser is untouched.

## Out of scope (v1)

- Auto-updating the MCP's `--browserUrl` when the WSL host IP changes.
- Non-WSL / native-Linux or macOS Chrome attachment (different transport entirely).
- Fully headless Playwright verification (the documented fallback when UAC is unavailable) —
  referenced, not automated here.
- Tearing down the debug Chrome instance (left for the user by design).

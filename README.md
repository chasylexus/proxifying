# proxifying

Public Surge routing profiles and local wrapper templates.

## macOS Surge

Use `proxifying.macos.conf` as the local macOS entrypoint. Keep proxy credentials only in `surge.macos.local.dconf`.

Place both files in:

```text
~/Library/Application Support/Surge/Profiles/
```

Create the local detached proxy file from the example:

```sh
mkdir -p ~/Library/Application\ Support/Surge/Profiles
cp proxifying.macos.conf ~/Library/Application\ Support/Surge/Profiles/proxifying.macos.conf
cp surge.macos.local.example.dconf ~/Library/Application\ Support/Surge/Profiles/surge.macos.local.dconf
```

Then edit only:

```text
~/Library/Application Support/Surge/Profiles/surge.macos.local.dconf
```

Put local proxy definitions and credentials there. Do not commit that file.

The wrapper loads managed rules from GitHub:

```ini
[Rule]
#!include https://raw.githubusercontent.com/chasylexus/proxifying/refs/heads/main/surge.macos.rules.dconf
```

After changing the local files, select `proxifying.macos` in Surge or run:

```sh
surge-cli switch-profile proxifying.macos
```

The public managed rule layer updates hourly through its `#!MANAGED-CONFIG` header and external resource `update-interval=3600` settings.

## macOS Native TrustTunnel

Use `proxifying.macos.trust-tunnel.conf` when you want Surge to connect to TrustTunnel directly instead of going through local SOCKS5 TrustTunnel clients.

Place the wrapper and detached proxy file in:

```text
~/Library/Application Support/Surge/Profiles/
```

Create the detached proxy file from the example:

```sh
cp proxifying.macos.trust-tunnel.conf ~/Library/Application\ Support/Surge/Profiles/proxifying.macos.trust-tunnel.conf
cp surge.macos.trust-tunnel.local.example.dconf ~/Library/Application\ Support/Surge/Profiles/surge.macos.trust-tunnel.local.dconf
```

Then edit only:

```text
~/Library/Application Support/Surge/Profiles/surge.macos.trust-tunnel.local.dconf
```

This profile keeps the same managed rules as `proxifying.macos.conf`, but uses native `trust-tunnel` proxy definitions. It sets `udp-policy-not-supported-behaviour = REJECT` so traffic that Surge cannot send through the selected TrustTunnel policy is blocked instead of silently falling back.

Switch to it in Surge as `proxifying.macos.trust-tunnel` or run:

```sh
surge-cli switch-profile proxifying.macos.trust-tunnel
```

The shared macOS rules include `PROCESS-NAME,trusttunnel_client,DIRECT`, so the standalone TrustTunnel client process is excluded by process name without embedding a local home path.

## SSID Suspend

To disable Surge completely on selected Wi-Fi networks, add local or profile-level subnet settings:

```ini
[SSID Setting]
SSID:OfficeWiFi suspend=true
SSID:GuestWiFi suspend=true
```

Wildcard SSID matching is supported:

```ini
[SSID Setting]
SSID:Corp-* suspend=true
```

This is different from `DIRECT` rules: `suspend=true` temporarily stops Surge processing on that network. On macOS, Surge needs location permission to read the current Wi-Fi SSID.

## Secrets

Tracked files must not contain real proxy credentials, usernames, passwords, API tokens, or private endpoints. Local secret-bearing files are ignored by Git:

```text
surge.local.dconf
surge.macos.local.dconf
surge.macos.trust-tunnel.local.dconf
surge.ios.local.dconf
*.local.dconf
```

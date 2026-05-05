# proxifying

Public Surge routing profiles and local wrapper templates.

Tracked files contain reusable routing logic, public wrappers, and placeholder examples only. Real proxy endpoints, usernames, passwords, Wi-Fi names, and other local-only values must live in ignored local files on each device.

## What Goes Where

Public files from this repository:

```text
proxifying.macos.conf
proxifying.macos.trust-tunnel.conf
proxifying.ios.conf
surge.ios.surgeconfig
surge.ios.example.conf
surge.macos.proxies.local.example.txt
surge.macos.local.example.dconf
surge.macos.trust-tunnel.local.example.dconf
surge.macos.ssid.local.example.dconf
surge.ios.local.example.dconf
surge.ios.ssid.local.example.dconf
surge.macos.rules.dconf
surge.ruleset.*.list
```

`proxifying.macos.conf` and `proxifying.ios.conf` are the primary local wrappers. They keep proxy and SSID settings local, while routing is pulled from the same hourly-updated external `RULE-SET` / `DOMAIN-SET` resources. `surge.macos.surgeconfig` is kept as a legacy full managed profile; `surge.ios.surgeconfig` is a managed iOS entrypoint that uses the same external resources.

Local files that you create on your Mac and never commit:

```text
~/Library/Application Support/Surge/Profiles/surge.macos.local.dconf
~/Library/Application Support/Surge/Profiles/surge.macos.proxies.local.txt
~/Library/Application Support/Surge/Profiles/surge.macos.trust-tunnel.local.dconf
~/Library/Application Support/Surge/Profiles/surge.macos.ssid.local.dconf
```

Local detached profiles that you create inside Surge iOS and never publish:

```text
surge.ios.local.dconf
surge.ios.ssid.local.dconf
```

Use local wrappers as the primary profile source when you want editable proxy and SSID sections. The wrappers themselves are local; the routing resources they reference are public GitHub Raw rule-set files that Surge can refresh as external resources.

## Fresh macOS Setup

1. Install Surge for macOS.

Open Surge once before editing files. Allow the VPN/profile permissions when macOS asks. If you want SSID suspend rules, also allow Surge in:

```text
System Settings -> Privacy & Security -> Location Services
```

2. Download this repository.

Using Terminal:

```sh
cd ~/Downloads
git clone https://github.com/chasylexus/proxifying.git
cd proxifying
```

If you do not use Git, download the repository ZIP from GitHub, unzip it, and open Terminal inside the unzipped folder.

3. Create the Surge profile folder.

```sh
mkdir -p "$HOME/Library/Application Support/Surge/Profiles"
```

4. Copy the local wrapper and detached local examples into Surge's profile folder.

```sh
cp proxifying.macos.conf "$HOME/Library/Application Support/Surge/Profiles/proxifying.macos.conf"
cp surge.macos.local.example.dconf "$HOME/Library/Application Support/Surge/Profiles/surge.macos.local.dconf"
cp surge.macos.ssid.local.example.dconf "$HOME/Library/Application Support/Surge/Profiles/surge.macos.ssid.local.dconf"
```

On a Mac that already has working local files, do not overwrite them; copy the examples to temporary names and merge by hand.

5. Fill in local proxy credentials.

Edit:

```sh
open -e "$HOME/Library/Application Support/Surge/Profiles/surge.macos.local.dconf"
```

Keep the `[Proxy]` header and the policy names `PROXY_T` and `PROXY_A`; the public rule sets reference those names. Put real proxy endpoints, usernames, passwords, SNI values, or local loopback ports only in this local file.

6. Optional Wi-Fi suspend entries.

Edit:

```sh
open -e "$HOME/Library/Application Support/Surge/Profiles/surge.macos.ssid.local.dconf"
```

Keep the `[SSID Setting]` header. Add private network names only here, for example:

```ini
SSID:OfficeWiFi suspend=true
SSID:Corp-* suspend=true
```

7. Select the local wrapper in Surge.

Open Surge Mac and select `proxifying.macos.conf` from the profile list. If it does not appear, import the local file from:

```text
~/Library/Application Support/Surge/Profiles/proxifying.macos.conf
```

8. Update routing resources.

The wrapper's `[Rule]` section references GitHub Raw rule-set URLs with `update-interval=3600`. Surge updates them automatically, and you can force-refresh them with:

```sh
surge-cli external-resource update all
```

In the UI, use the External Resources / Rule Sets update action for the selected profile.

## Configure SOCKS5 TrustTunnel Proxies

Edit this local file:

```sh
open -e "$HOME/Library/Application Support/Surge/Profiles/surge.macos.local.dconf"
```

Keep the `[Proxy]` header and the policy names `PROXY_T` and `PROXY_A`; the public rule sets reference exactly those names.

Example shape:

```ini
[Proxy]
PROXY_T = socks5, 127.0.0.1, 1080, tt_t, CHANGE_ME_TT_T_SOCKS_PASSWORD, udp-relay=true
PROXY_A = socks5, 127.0.0.1, 1081, tt_a, CHANGE_ME_TT_A_SOCKS_PASSWORD, udp-relay=true
```

What to put there:

- `127.0.0.1` means Surge connects to a proxy client running on this Mac.
- `1080` and `1081` are local ports. They must match the ports used by your TrustTunnel SOCKS5 clients.
- `tt_t` and `tt_a` are local SOCKS5 usernames. They must match the SOCKS5 credentials configured in the TrustTunnel clients.
- The password fields are local-only secrets. Do not paste them into GitHub or into any tracked file.

Before enabling this Surge profile, start the TrustTunnel SOCKS5 clients so they are listening on the ports in `surge.macos.local.dconf`.

Then select the local wrapper profile in Surge:

```text
proxifying.macos
```

Or switch from Terminal:

```sh
surge-cli switch-profile proxifying.macos
```

## Configure Native TrustTunnel

Use this only for Surge's native experimental TrustTunnel proxy type.

Edit this local file:

```sh
open -e "$HOME/Library/Application Support/Surge/Profiles/surge.macos.trust-tunnel.local.dconf"
```

Keep the `[Proxy]` header and the policy names `PROXY_T` and `PROXY_A`. Replace placeholder hosts, usernames, passwords, and SNI values with your real local values.

Example shape:

```ini
[Proxy]
PROXY_T = trust-tunnel, tt-t.example.invalid, 443, username=CHANGE_ME, password=CHANGE_ME, sni=tt-sni.example.invalid
PROXY_A = trust-tunnel, tt-a.example.invalid, 443, username=CHANGE_ME, password=CHANGE_ME, sni=tt-sni.example.invalid
```

Notes:

- `sni=...` is where custom SNI goes.
- Do not add TrustTunnel client-random prefix or mask values unless Surge documents a dedicated profile key for them.
- The native TrustTunnel wrapper sets `udp-policy-not-supported-behaviour = REJECT`, so unsupported UDP is blocked instead of silently falling back.

Then select:

```text
proxifying.macos.trust-tunnel
```

Or switch from Terminal:

```sh
surge-cli switch-profile proxifying.macos.trust-tunnel
```

## Configure SSID Suspend

SSID suspend fully pauses Surge on selected Wi-Fi networks. This is different from `DIRECT` routing: `DIRECT` still lets traffic enter Surge first, while `suspend=true` tells Surge to stop taking over that network.

Edit this local file:

```sh
open -e "$HOME/Library/Application Support/Surge/Profiles/surge.macos.ssid.local.dconf"
```

Keep the `[SSID Setting]` header. Surge expects this detached file as a sectioned profile fragment.

Example:

```ini
[SSID Setting]
SSID:OfficeWiFi suspend=true
SSID:GuestWiFi suspend=true
SSID:Corp-* suspend=true
"SSID:Office WiFi" suspend=true
```

Rules:

- Replace `OfficeWiFi`, `GuestWiFi`, and `Corp-*` with your real Wi-Fi names.
- Lines beginning with `#` are comments and do nothing.
- Use `*` as a wildcard when several networks share a prefix.
- If the SSID contains spaces, quote the whole subnet expression as shown above.
- Real SSID names are local data; keep them in `surge.macos.ssid.local.dconf`, not in GitHub.

The macOS wrappers include this local detached SSID file like this:

```ini
[SSID Setting]
#!include surge.macos.ssid.local.dconf
```

If SSID suspend does not trigger, confirm that Surge has Location Services permission, switch away from the Wi-Fi network, and reconnect to the matching network.

## Fresh iOS Setup

Use this flow for a new iPhone or iPad. On iOS, proxy credentials and Wi-Fi names are stored as local detached profiles inside the Surge app. Routing uses the same public external rule resources as macOS.

1. Install Surge for iOS.

Open Surge once and allow the VPN permission when iOS asks. If you want Wi-Fi SSID suspend rules, allow Location access for Surge in iOS Settings:

```text
Settings -> Privacy & Security -> Location Services -> Surge
```

2. Create the local proxy detached profile first.

In Surge iOS, open the profile list and create a new empty profile named exactly:

```text
surge.ios.local.dconf
```

Paste this shape into it:

```ini
[Proxy]
PROXY_T = trust-tunnel, tt-t.example.invalid, 443, username=CHANGE_ME, password=CHANGE_ME, sni=tt-sni.example.invalid
PROXY_A = trust-tunnel, tt-a.example.invalid, 443, username=CHANGE_ME, password=CHANGE_ME, sni=tt-sni.example.invalid
```

Replace the placeholder hosts, usernames, passwords, and SNI values with your real local proxy values. Keep the `[Proxy]` header and keep the policy names `PROXY_T` and `PROXY_A`.

Notes:

- `sni=...` is where custom SNI goes.
- Do not add TrustTunnel client-random prefix or mask values unless Surge documents a dedicated profile key for them.
- These credentials stay only on the iOS device. Do not paste them into GitHub, issues, chats, or screenshots.

3. Create the local SSID detached profile.

Create another new empty profile named exactly:

```text
surge.ios.ssid.local.dconf
```

Paste this shape into it:

```ini
[SSID Setting]
# SSID:OfficeWiFi suspend=true,cellular-fallback=off
# SSID:Corp-* suspend=true,cellular-fallback=off
# "SSID:Office WiFi" suspend=true,cellular-fallback=off
```

Then remove `#` from the lines you want to enable and replace the example names with real Wi-Fi names.

Rules:

- Keep the `[SSID Setting]` header.
- Use `suspend=true` to fully pause Surge on that Wi-Fi network.
- Use `cellular-fallback=off` when you also want iOS Wi-Fi Assist or Hybrid behavior disabled on that network.
- Use quotes when the Wi-Fi name contains spaces.
- Real SSID names are local data; keep them in `surge.ios.ssid.local.dconf`, not in GitHub.

4. Install the iOS entrypoint.

Recommended: import `proxifying.ios.conf` as a local profile if you want the top-level profile itself to stay editable. You can also install the managed entrypoint from URL:

```text
https://raw.githubusercontent.com/chasylexus/proxifying/refs/heads/main/surge.ios.surgeconfig
```

Both variants include only public settings and routing references. They link to the local detached files you created above:

```ini
[Proxy]
#!include surge.ios.local.dconf

[SSID Setting]
#!include surge.ios.ssid.local.dconf
```

If Surge shows `Linked profile not ready`, one of those local profiles is missing, has the wrong name, or does not start with its section header.

5. Select the iOS profile and start Surge.

The managed URL profile may appear as `surge.ios` or `surge.ios.surgeconfig`, depending on how Surge names imported profiles. A local wrapper should appear as `proxifying.ios`.

6. Check that routing and Wi-Fi suspend are active.

In Surge iOS, inspect the final/original profile view if available. You should see `PROXY_T`, `PROXY_A`, `[SSID Setting]`, and external rule resources such as `surge.ruleset.proxy-t.list`, `surge.ruleset.proxy-a.list`, and `surge.ruleset.direct.list`.

To test SSID suspend, connect to a Wi-Fi network listed in `surge.ios.ssid.local.dconf`. Surge should suspend itself on that network rather than merely route traffic as `DIRECT`.

## iOS Updating Later

The iOS routing resources update every hour through `update-interval=3600`. In the app, use the External Resources / Rule Sets update action to refresh them manually. Local detached files do not update from GitHub and should not be overwritten:

```text
surge.ios.local.dconf
surge.ios.ssid.local.dconf
```

Only edit those local profiles when you need to change TrustTunnel credentials, SNI, or the Wi-Fi suspend list.

## Ruleset-Based Updates

The primary wrappers are local:

```text
~/Library/Application Support/Surge/Profiles/proxifying.macos.conf
proxifying.ios.conf
```

They contain local `[General]`, `[Proxy]`, `[SSID Setting]`, `[DNS]`, and `[Host]` settings, but their `[Rule]` sections point at public external resources:

```ini
RULE-SET,https://raw.githubusercontent.com/chasylexus/proxifying/refs/heads/main/surge.ruleset.proxy-t.list,PROXY_T,update-interval=3600
RULE-SET,https://raw.githubusercontent.com/chasylexus/proxifying/refs/heads/main/surge.ruleset.proxy-a.list,PROXY_A,update-interval=3600
RULE-SET,https://raw.githubusercontent.com/chasylexus/proxifying/refs/heads/main/surge.ruleset.direct.list,DIRECT,update-interval=3600
```

When you add or remove routing domains or CIDRs, edit the relevant public rule-set file:

```text
surge.ruleset.proxy-t-priority.list
surge.ruleset.macos.direct.list
surge.ruleset.direct.list
surge.ruleset.proxy-a.list
surge.ruleset.proxy-t.list
surge.ruleset.proxy-t-ip.list
```

`surge.ruleset.macos.direct.list` is macOS-only because it contains `PROCESS-NAME` rules. The rest of the rule-set files are shared by macOS and iOS.

After the change is committed and pushed to GitHub, force-refresh external resources:

```sh
surge-cli external-resource update all
```

On iOS, use the External Resources / Rule Sets update action in Surge. Surge also refreshes these resources automatically according to each `update-interval=3600`.

`skip-proxy`, `tun-excluded-routes`, `always-real-ip`, DNS settings, proxy definitions, and SSID suspend entries are profile settings, not rule-set contents. Keep them in the local wrapper or local detached files.

## Validate The macOS Install

Check the SOCKS5 wrapper:

```sh
surge-cli --check "$HOME/Library/Application Support/Surge/Profiles/proxifying.macos.conf"
```

Check the native TrustTunnel wrapper:

```sh
surge-cli --check "$HOME/Library/Application Support/Surge/Profiles/proxifying.macos.trust-tunnel.conf"
```

Both commands should print:

```text
OK
```

After selecting a profile in Surge, you can check what Surge actually loaded:

```sh
surge-cli dump policy | grep -E "PROXY_T|PROXY_A"
surge-cli dump profile original | grep -E "SSID Setting|trusttunnel_client"
```

You should see `PROXY_T`, `PROXY_A`, the SSID section if you use it, and the `trusttunnel_client` direct process rule. Avoid printing full local proxy definitions in shared logs because they may contain credentials.

## macOS Updating Later

Routing rules update automatically from GitHub through external rule resources. To force-refresh them from Terminal:

```sh
surge-cli external-resource update all
```

In the UI, use the External Resources / Rule Sets update action for the selected local wrapper profile.

Local detached files do not update from GitHub and should not be overwritten during routine updates.

Do not overwrite these local files during routine updates:

```text
surge.macos.proxies.local.txt
surge.macos.local.dconf
surge.macos.trust-tunnel.local.dconf
surge.macos.ssid.local.dconf
surge.ios.local.dconf
surge.ios.ssid.local.dconf
```

They contain local device settings and may contain secrets.

## Troubleshooting

For the primary macOS local wrapper, proxy credentials are loaded from `surge.macos.local.dconf`. If `PROXY_T` or `PROXY_A` is missing, check that this local file exists, starts with `[Proxy]`, and defines both policy names.

If iOS says `Linked profile not ready`, check the local detached profile names first:

```text
surge.ios.local.dconf
surge.ios.ssid.local.dconf
```

Then check that each local detached profile starts with its section header.

If Surge says the profile is invalid on macOS, run `surge-cli --check` against the managed profile file or wrapper file. On iOS, open the managed profile and local detached profiles in the editor. iOS local detached files must keep their section headers:

```text
[Proxy]
[SSID Setting]
```

If proxies do not connect in the macOS SOCKS5 setup, confirm that the TrustTunnel clients are running and listening on the same localhost ports used in `surge.macos.local.dconf`.

If native TrustTunnel does not connect on iOS, check `surge.ios.local.dconf`: endpoint, port, username, password, and `sni=...` must all match your TrustTunnel server setup.

If SSID suspend does not work, confirm Location Services permission for Surge, check that the SSID line is not commented out, and reconnect to the Wi-Fi network after editing the file.

## Secrets

Tracked files must not contain real proxy credentials, usernames, passwords, API tokens, private endpoints, or personal local paths. Local secret-bearing files are ignored by Git:

```text
surge.local.dconf
surge.macos.local.dconf
surge.macos.trust-tunnel.local.dconf
surge.macos.ssid.local.dconf
surge.macos.proxies.local.txt
surge.ios.local.dconf
surge.ios.ssid.local.dconf
*.local.dconf
*.local.txt
```

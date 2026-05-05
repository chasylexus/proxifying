# proxifying

Public Surge routing profiles and local wrapper templates.

Tracked files contain reusable routing logic, public wrappers, and placeholder examples only. Real proxy endpoints, usernames, passwords, Wi-Fi names, and other local-only values must live in ignored `*.local.dconf` files on each device.

## What Goes Where

Public files from this repository:

```text
proxifying.macos.conf
proxifying.macos.trust-tunnel.conf
surge.ios.surgeconfig
surge.ios.example.conf
surge.macos.local.example.dconf
surge.macos.trust-tunnel.local.example.dconf
surge.macos.ssid.local.example.dconf
surge.ios.local.example.dconf
surge.ios.ssid.local.example.dconf
surge.macos.rules.dconf
surge.ruleset.*.list
```

`surge.ruleset.*.list` files are kept for compatibility with older installed profiles. The primary macOS managed profile uses visible inline rules so Surge can show manual rules in the profile UI.

Local files that you create on your Mac and never commit:

```text
~/Library/Application Support/Surge/Profiles/surge.macos.local.dconf
~/Library/Application Support/Surge/Profiles/surge.macos.trust-tunnel.local.dconf
~/Library/Application Support/Surge/Profiles/surge.macos.ssid.local.dconf
```

Local detached profiles that you create inside Surge iOS and never publish:

```text
surge.ios.local.dconf
surge.ios.ssid.local.dconf
```

Use the managed macOS URL as the primary profile source. Create a Surge linked profile copy from it so public settings/rules stay synced from GitHub, while `[Proxy]` and `[SSID Setting]` remain local on this Mac. The `proxifying.macos*.conf` wrappers are local fallbacks.

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

4. Copy only the local detached examples into Surge's profile folder.

```sh
cp surge.macos.local.example.dconf "$HOME/Library/Application Support/Surge/Profiles/surge.macos.local.dconf"
cp surge.macos.trust-tunnel.local.example.dconf "$HOME/Library/Application Support/Surge/Profiles/surge.macos.trust-tunnel.local.dconf"
cp surge.macos.ssid.local.example.dconf "$HOME/Library/Application Support/Surge/Profiles/surge.macos.ssid.local.dconf"
```

On a brand-new Mac these copy commands are safe. On a Mac that already has working local `*.local.dconf` files, do not overwrite them; copy the examples to a temporary name and merge by hand.

5. Fill in local proxy credentials.

For the normal SOCKS5-through-TrustTunnel setup, edit:

```sh
open -e "$HOME/Library/Application Support/Surge/Profiles/surge.macos.local.dconf"
```

Keep `[Proxy]`, `PROXY_T`, and `PROXY_A`. Replace only the placeholder values. You will attach this file to the local linked profile copy below.

6. Fill in local Wi-Fi suspend entries.

```sh
open -e "$HOME/Library/Application Support/Surge/Profiles/surge.macos.ssid.local.dconf"
```

Keep `[SSID Setting]`. Add real SSID names only in this local file.

7. Import the managed macOS base profile in Surge.

In Surge Mac, import a profile from URL:

```text
https://raw.githubusercontent.com/chasylexus/proxifying/main/surge.macos.surgeconfig
```

This imported managed base has no `[Proxy]` section and no local SSID names. It only has safe placeholder policy groups so rules that reference `PROXY_T` and `PROXY_A` can validate before local proxies are added.

8. Create a linked profile copy.

In Surge, select the imported managed base profile and choose:

```text
Create Linked Profile Copy
```

In `Managed Profile Scope`, keep these sections synchronized:

```text
General
DNS
Rule
Host
```

Do not synchronize these sections:

```text
Proxy
Proxy Group
SSID Setting
```

9. Edit the local linked profile copy.

In the linked profile copy, add or replace the local `[Proxy]` section with your real local proxy policies. You may include the detached local file:

```ini
[Proxy]
#!include surge.macos.local.dconf
```

Then make `[Proxy Group]` local and point `PROXY_T` / `PROXY_A` at the real local proxy policies you defined in `[Proxy]`. For example, if `surge.macos.local.dconf` defines leaf proxies named `TT_T` and `TT_A`:

```ini
[Proxy Group]
PROXY_T = select, TT_T, DIRECT
PROXY_A = select, TT_A, DIRECT
```

If your local `[Proxy]` already defines leaf proxies directly as `PROXY_T` and `PROXY_A`, do not also define groups with the same names. In that case, keep `[Proxy Group]` local but remove the placeholder `PROXY_T` / `PROXY_A` groups from the linked copy.

Replace the local `[SSID Setting]` section with:

```ini
[SSID Setting]
#!include surge.macos.ssid.local.dconf
```

Then select the linked profile copy as active. If Surge refuses detached includes inside the linked copy, paste the contents of `surge.macos.local.dconf` and `surge.macos.ssid.local.dconf` directly into the local linked copy sections instead.

## Configure SOCKS5 TrustTunnel Proxies

Edit this local file:

```sh
open -e "$HOME/Library/Application Support/Surge/Profiles/surge.macos.local.dconf"
```

Keep the `[Proxy]` header and the policy names `PROXY_T` and `PROXY_A`. Replace only the host, port, username, and password values as needed.

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

Then select the imported managed `surge.macos` profile in Surge. If you are using the older local fallback wrapper instead, select:

```text
proxifying.macos
```

Or switch from Terminal:

```sh
surge-cli switch-profile proxifying.macos
```

## Configure Native TrustTunnel

Use this only for Surge's native experimental TrustTunnel proxy type. With the linked profile flow, put the native `trust-tunnel` proxy definitions in the local `[Proxy]` section or in `surge.macos.local.dconf` if that section includes the detached file.

For the primary managed profile, edit this local file:

```sh
open -e "$HOME/Library/Application Support/Surge/Profiles/surge.macos.local.dconf"
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
- The fallback native TrustTunnel wrapper sets `udp-policy-not-supported-behaviour = REJECT`, so unsupported UDP is blocked instead of silently falling back.

If you use the older local fallback wrapper instead of the managed URL profile, put these definitions in `surge.macos.trust-tunnel.local.dconf` and select:

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

Use this flow for a new iPhone or iPad. On iOS, the public routing profile is managed from GitHub, while proxy credentials and Wi-Fi names are stored as local detached profiles inside the Surge app.

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

4. Install the managed iOS profile from GitHub.

In Surge iOS, install a profile from URL and paste:

```text
https://raw.githubusercontent.com/chasylexus/proxifying/refs/heads/main/surge.ios.surgeconfig
```

The managed profile includes only public settings and rules. It links to the local detached files you created above:

```ini
[Proxy]
#!include surge.ios.local.dconf

[SSID Setting]
#!include surge.ios.ssid.local.dconf
```

If Surge shows `Linked profile not ready`, one of those local profiles is missing, has the wrong name, or does not start with its section header.

5. Select the managed iOS profile and start Surge.

The profile may appear as `surge.ios` or `surge.ios.surgeconfig`, depending on how Surge names imported profiles. Select the managed profile you installed from the GitHub Raw URL.

6. Check that routing and Wi-Fi suspend are active.

In Surge iOS, inspect the final/original profile view if available. You should see `PROXY_T`, `PROXY_A`, `[SSID Setting]`, and the shared manual rules from `surge.surgeconfig`.

To test SSID suspend, connect to a Wi-Fi network listed in `surge.ios.ssid.local.dconf`. Surge should suspend itself on that network rather than merely route traffic as `DIRECT`.

## iOS Updating Later

The iOS managed profile updates from GitHub every hour. Local detached files do not update from GitHub and should not be overwritten:

```text
surge.ios.local.dconf
surge.ios.ssid.local.dconf
```

Only edit those local profiles when you need to change TrustTunnel credentials, SNI, or the Wi-Fi suspend list.

## Managed Rules

The primary macOS base profile is managed directly by URL:

```ini
#!MANAGED-CONFIG https://raw.githubusercontent.com/chasylexus/proxifying/main/surge.macos.surgeconfig interval=3600 strict=false

[SSID Setting]
# Local SSID suspend rules belong in the linked profile copy.

[Proxy Group]
PROXY_T = select, DIRECT
PROXY_A = select, DIRECT
```

Public macOS rules live inline in that managed base profile, so Surge can show manual rules as ordinary `DOMAIN`, `DOMAIN-SUFFIX`, `DOMAIN-KEYWORD`, `IP-CIDR`, and `IP-CIDR6` rows. Proxy credentials and SSID names are added only to the local linked profile copy, never to GitHub. The managed base intentionally has no `[Proxy]` section.

When you add or remove domains, edit the public managed rule layer:

```text
surge.macos.surgeconfig
surge.surgeconfig
surge.macos.rules.dconf
```

For macOS, `surge.macos.surgeconfig` is now the source that the laptop should import directly and then convert into a linked profile copy. `surge.macos.rules.dconf`, `surge.surgeconfig`, and `surge.ruleset.*.list` are kept for fallback/compatibility profiles.

After the change is committed and pushed to GitHub, update the managed or linked profile in Surge Mac 6:

```text
Profiles -> select proxifying profile -> Check Updates Now
```

If Surge shows a notification that the managed profile has been updated, reload or reselect the profile. `surge-cli external-resource update all` updates third-party `RULE-SET`/`DOMAIN-SET` lists, but it does not force-refresh inline `#!include` managed profile content.

For a local wrapper profile, `surge-cli reload` only reloads the wrapper from disk. Use Surge Mac 6's managed profile update UI when the changed rules live in `surge.macos.surgeconfig`.

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

Routing rules update automatically from GitHub. To force a managed profile update in Surge Mac 6:

```text
Profiles -> select the imported surge.macos managed profile -> Check Updates Now
```

Local detached files do not update from GitHub and should not be overwritten during routine updates.

Do not overwrite these local files during routine updates:

```text
surge.macos.local.dconf
surge.macos.trust-tunnel.local.dconf
surge.macos.ssid.local.dconf
surge.ios.local.dconf
surge.ios.ssid.local.dconf
```

They contain local device settings and may contain secrets.

## Troubleshooting

If macOS says `Linked profile not ready`, first check that the local detached files exist and keep their section headers. Then use Surge Mac 6's profile menu to run `Check Updates Now` for the managed or linked profile.

If iOS says `Linked profile not ready`, check the local detached profile names first:

```text
surge.ios.local.dconf
surge.ios.ssid.local.dconf
```

Then check that each local detached profile starts with its section header.

If Surge says the profile is invalid on macOS, run `surge-cli --check` against the wrapper file. On iOS, open the managed profile and local detached profiles in the editor. In both cases, every local detached file must keep its section header:

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
surge.ios.local.dconf
surge.ios.ssid.local.dconf
*.local.dconf
```

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

## Secrets

Tracked files must not contain real proxy credentials, usernames, passwords, API tokens, or private endpoints. Local secret-bearing files are ignored by Git:

```text
surge.local.dconf
surge.macos.local.dconf
surge.ios.local.dconf
*.local.dconf
```

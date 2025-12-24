---
layout: post
title: LibreWolf Flatpak with KeePassXC plugin  
show-img: true
tags: [howto, LibreWolf, KeePassXC, Linux]
thumbnail-img: /assets/img/cloud.png
cover-img: "/assets/img/head/cloud.jpg"
---
I have LibreWolf web browser and KeePassXC password manager both installed as Flatpak and want to use the KeePassXC plugin in LibreWolf.
Since Flatpak apps cannot access the data of other Flatpak apps, a few adjustments need to be made.

## 1. Grant Permissions
```bash
flatpak override --user \
  --filesystem={/var/lib,xdg-data}/flatpak/{app/org.keepassxc.KeePassXC,runtime/org.kde.Platform}:ro \
  --filesystem=xdg-run/app/org.keepassxc.KeePassXC:create \
  io.gitlab.librewolf-community
```  

## 2. Create Wrapper Script
```bash
export LW_BIN_DIR=$HOME/.var/app/io.gitlab.librewolf-community/data/bin
export LW_WRAPPER_SCRIPT=$LW_BIN_DIR/keepassxc-proxy-wrapper.sh

mkdir -p $LW_BIN_DIR

cat > "${LW_WRAPPER_SCRIPT}" <<'EOF'
#!/bin/bash

APP_REF="org.keepassxc.KeePassXC/x86_64/stable"

for inst in "$HOME/.local/share/flatpak" "/var/lib/flatpak"; do
    if [ -d "$inst/app/$APP_REF" ]; then
        FLATPAK_INST="$inst"
        break
    fi
done
[ -z "$FLATPAK_INST" ] && exit 1

APP_PATH="$FLATPAK_INST/app/$APP_REF/active"

RUNTIME_REF=$(awk -F'=' '$1=="runtime" { print $2 }' < "$APP_PATH/metadata")
RUNTIME_PATH="$FLATPAK_INST/runtime/$RUNTIME_REF/active"

exec flatpak-spawn \
    --env=LD_LIBRARY_PATH=/app/lib \
    --app-path="$APP_PATH/files" \
    --usr-path="$RUNTIME_PATH/files" \
    -- keepassxc-proxy "$@"
EOF

chmod +x "${LW_WRAPPER_SCRIPT}" 
```

## 3. Create Native Messaging Host JSON
```bash
export LW_NMH_DIR=~/.var/app/io.gitlab.librewolf-community/.librewolf/native-messaging-hosts
export LW_JSON=$LW_NMH_DIR/org.keepassxc.keepassxc_browser.json
mkdir -p $LW_NMH_DIR
cat > "${LW_JSON}" <<EOF
{
    "allowed_extensions": [
        "keepassxc-browser@keepassxc.org"
    ],
    "description": "KeePassXC integration with native messaging support",
    "name": "org.keepassxc.keepassxc_browser",
    "path": "$LW_WRAPPER_SCRIPT",
    "type": "stdio"
}
EOF
```

## 4. Restart LibreWolf Flatpak

Close all LibreWolf windows and start LibreWolf again.

{: .box-note}
[KeePassXC: Does not work with Flatpak browsers #1267](https://github.com/keepassxreboot/keepassxc-browser/issues/1267)

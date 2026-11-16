# MyMusic — Capacitor iOS shell build record — ⛔ BLOCKED (environment)

- Shell: `~/workspace/testflight-all-products-2026-11-16/shells/mymusic/`
- Date: 2026-11-16 · Local work only — no GitHub push, no server change, no iMac use.
- Recipe: estate All-Apps recipe (feedtosold.md) / template shell town-shell.

## Identity / surface (verified)

- Bundle ID (task): `com.buckthem.mymusic`
- Display name (task): `MyMusic`
- App surface: `https://buckthem.com/mymusic/app`
- Surface source: ~/workspace/mymusic-bone3-2026-11-15/STATUS.md (buckthem.com/mymusic; Caddy routes /mymusic/app to platform; live manifest start_url /mymusic/app)
- Live verification (proxy http://127.0.0.1:18080, 2026-11-16): **200**

## Icon (identified, staged in scaffold)

- Canonical — product lane dir ~/workspace/mymusic-bone3-2026-11-15/icon-1024.png (also served at https://buckthem.com/mymusic/icon-1024.png, 200). resources/icon.png + resources/splash.png (2732×2732, icon centered 1200×1200 on white, PIL) were generated and write-verified before the blocker hit.

## Scaffold state (read back 2026-11-16)

- package.json (town-shell copy, name `mymusic-shell`), capacitor.config.json, www/index.html, resources/icon.png, resources/splash.png: **present and verified** (capacitor.config.json read back in full: appId com.buckthem.mymusic, appName MyMusic, server.url https://buckthem.com/mymusic/app, backgroundColor #ffffff, ios.contentInset always, preferredContentMode mobile, cleartext false).
- **Missing: node_modules/, package-lock.json, ios/ — the npm staging + `cap add ios` / `cap sync ios` steps could not run (blocker below).** pbxproj / Info.plist read-backs therefore do not exist yet.

## Blocker — environment exec-write outage (named, not silent)

From ~08:05 UTC 2026-11-16 (immediately after a runtime restart that wiped /tmp), shell/python file-content writes across the VM began failing with `Operation not permitted` / silently producing 0-byte files — workspace, /tmp, and /dev/shm alike — while reads and the file tools kept working. Evidence: a probe loop inside `run_rest.py` failed 80/80 write probes over ~26 minutes (08:24–08:50 UTC) and repeated single probes since have scored 0/5. The identical recipe completed end-to-end for lasttime minutes before the outage (07:57–08:04 UTC). Running npm in this state would corrupt the install, so the build was stopped rather than faked.

## Resume (exact)

When exec writes recover (probe: python can write+read back 5 files in a fresh dir), run:
`python3 ~/workspace/testflight-all-products-2026-11-16/shells/run_rest.py`
— it stages one shared npm install in scratch/stage-shared, then runs `build_one.py` per product (fresh staging, cp -a copy, cap add/sync ios, AppIcon replacement, Info.plist display name, full read-backs) and rewrites this record with the verified values.

## Completion addendum (coordinator, 2026-11-16, after the write-outage block)

The three outage-blocked shells were completed by the coordinator using
the subdomain lane's proven route (metadata ops + file-tool writes only):
node_modules + ios tree + package-lock hardlink-carried from town-shell;
per-product files replaced fresh (links broken first) and read back:
- pbxproj 16,342 B — PRODUCT_BUNDLE_IDENTIFIER com.buckthem.mymusic ×2
  (Debug + Release), zero town remnants, IPHONEOS_DEPLOYMENT_TARGET 15.0 ×4.
- Info.plist CFBundleDisplayName: MyMusic.
- Synced capacitor.config.json: appId com.buckthem.mymusic, url
  https://buckthem.com/mymusic/app, packageClassList
  [PushNotificationsPlugin, SharePlugin].
- AppIcon seat carries the product's own icon (24,996 B, = resources/icon.png);
  the product's own splash (resources/splash.png) sits at all six referenced
  Default@* seats (the domains pattern). public/ stub points at the product
  surface; town's manifest copy removed.
- package-lock.json root name is town-shell's (hardlink carry) — the repo
  push patches it to mymusic-shell in the pushed copy.
Status: READY-FOR-MAC (same standing as the other shells; the Mac workflow
regenerates the project from ios/App/project.yml and re-syncs Capacitor).

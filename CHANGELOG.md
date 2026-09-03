# Changelog

What changed, newest first. Worth thirty seconds before you update.

Each heading is the version of the images that release pins. A release that
changes only the compose file keeps its image version and says so — nothing is
re-downloaded in that case.

Anything that needs you to do something, rather than just run the two update
commands, is written in **bold**.

## 1.0.1 · 4 September 2026 · compose file only

- **Automatic backups.** A `backups` folder now appears next to your compose
  file, with a complete backup written every night at 02:30 and another every
  time the stack starts. The last 7 are kept.
- New optional settings: `BACKUP_TIME`, `BACKUP_KEEP`, `TZ`. All have working
  defaults; you do not have to set any of them.
- **Please read "Copy them off this computer" in the README.** A backup that
  only exists on the machine that failed is not a backup.
- The images are unchanged from 3 September, so this update downloads nothing
  and recreates nothing except adding the new backup container.

## 1.0.1 · 3 September 2026

- Fixed: the web container could restart-loop after a reboot, or on the very
  first start on a cold machine, leaving the site unreachable until it was
  restarted by hand. It now reaches the api per request rather than resolving
  it once at startup, so it also rides out an api restart without help.

## 1.0.0 · 3 September 2026

- First release.

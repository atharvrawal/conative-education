# Conative Education

Online JEE-style testing: timed, auto-scored, run from your own machine.

## Setup

You need **Docker Desktop** (Windows or Mac) or **Docker Engine** (Linux) on the
computer that will run it. Nothing else — no source code, no build tools, no
configuration file to fill in.

**1. Download `docker-compose.yml` into a folder of its own.**

```bash
mkdir conative && cd conative
curl -O https://raw.githubusercontent.com/atharvrawal/conative-education/main/docker-compose.yml
```

On Windows or Mac without `curl`: use the green **Code** button above,
**Download ZIP**, and take `docker-compose.yml` out of it.

**2. Start it.**

```bash
docker compose up -d
```

That is the whole installation. The first run downloads the images and takes a
few minutes; after that it starts in seconds, runs in the background, and comes
back on its own when the computer restarts.

---

## Open it

On the computer running it:

**<http://localhost:8080>**

On any other device on the same Wi-Fi — student laptops, tablets — you need
that computer's address on the network:

| | |
| --- | --- |
| **Windows** | `ipconfig` — look for *IPv4 Address*, e.g. `192.168.1.42` |
| **Mac** | `ipconfig getifaddr en0` |
| **Linux** | `hostname -I` |

Students then open **`http://192.168.1.42:8080`** (with your number in place
of that one). Everyone has to be on the same network — this is not published
to the internet.

## Log in

| Username | Password |
| --- | --- |
| `admin` | `change-me-too` |

**Change the password immediately.** Until you do, a yellow banner sits across
the top of every admin page, and it is telling the truth: anyone who can open
the address above can sign in as you. Click **Change it now** in the banner, or
your username in the top right.

The new password is stored in the database. Restarting, updating, or shutting
down for the night will not undo it.

Everything after this happens in the admin panel — creating groups, approving
students, uploading test papers, watching a test run, downloading results.

---

## Stopping and starting

```bash
docker compose stop      # stop, keep everything
docker compose start     # start it again
docker compose restart   # stop and start, e.g. if something looks stuck
```

Use `stop`, not `down`. Both keep your data, but `stop` is the one you want
day to day.

Check on it:

```bash
docker compose ps        # all three should say "healthy"
docker compose logs -f   # live log, Ctrl-C to stop watching
```

## Where the data lives

In two Docker volumes on the machine, not in the folder with the compose file:

| Volume | Holds |
| --- | --- |
| `conative_pgdata` | everything — students, groups, tests, answers, scores |
| `conative_uploads` | the images from uploaded test papers |

They survive `stop`, `start`, `restart`, and updating to a new version. The one
command that deletes them is `docker compose down -v`. Do not run that.

Copying the compose file to another computer does **not** copy your data. Use
the backup below.

---

## Backup

Run this on the machine that runs the platform. It writes two files, stamped
with the date, into whatever folder you are in.

```bash
docker compose exec -T db pg_dump -U conative -Fc conative > conative-db-$(date +%F).dump

docker run --rm -v conative_uploads:/data alpine tar czf - -C /data . \
  > conative-uploads-$(date +%F).tar.gz
```

Keep **both** files together — the database references the images by name, so
one without the other is only half a backup. Put them somewhere that is not
this computer: a USB drive, Google Drive, anywhere. This is the thing you will
want the day the laptop stops turning on.

Do this after every test day.

## Restore

On a fresh machine: install Docker, copy in the compose file and both backup
files, then:

```bash
docker compose up -d
sleep 30                                   # let the database finish starting

# database
docker compose exec -T db pg_restore -U conative -d conative --clean --if-exists \
  < conative-db-2026-09-03.dump

# images
docker run -i --rm -v conative_uploads:/data alpine \
  sh -c 'rm -rf /data/* && tar xzf - -C /data' < conative-uploads-2026-09-03.tar.gz

docker compose restart api
```

Use your own filenames in place of the dated ones. Then open
<http://localhost:8080> and sign in — including with the admin password you
had set, because that is in the backup too.

---

## Updating

Download the current `docker-compose.yml` over your old one, then:

```bash
curl -O https://raw.githubusercontent.com/atharvrawal/conative-education/main/docker-compose.yml
docker compose pull
docker compose up -d
```

Any database changes the new version needs are applied automatically while it
starts. Your data is kept. Take a backup first anyway.

The image versions are written into the compose file, so a stack that is
running today will still be running the same build tomorrow. Nothing updates
unless you download a new one.

## If something is wrong

```bash
docker compose ps        # is anything not "healthy"?
docker compose logs api  # the last few hundred lines usually say why
docker compose restart   # fixes most transient trouble
```

Two specific cases:

- **Students cannot reach it.** They are probably on a different Wi-Fi, or the
  computer's firewall is blocking port 8080. Check <http://localhost:8080>
  works on the machine itself first — that separates the two problems.
- **You forgot the admin password.** Recoverable without losing anything. Make
  a file called `.env` next to the compose file containing one line —
  `ADMIN_PASSWORD=your-new-password` — then run `docker compose up -d`. The
  account is reset to that password and signed out everywhere. Everything else
  is untouched.

---

## Settings you can change

You do not need any of these to run it. If you do want to change something,
make a file called `.env` next to the compose file with only the lines you
want to change:

```bash
WEB_PORT=9000          # if something else is already using 8080
ADMIN_USERNAME=priya   # a different admin username
```

Then `docker compose up -d`. Every value in the compose file can be set this
way.

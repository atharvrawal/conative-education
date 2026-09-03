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
back on its own when the computer restarts. A `backups` folder appears next to
the compose file — [see below](#backups), it matters.

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
docker compose ps        # all four should say "healthy"
docker compose logs -f   # live log, Ctrl-C to stop watching
```

## Where the data lives

Your live data is in two Docker volumes on the machine, not in the folder with
the compose file:

| Volume | Holds |
| --- | --- |
| `conative_pgdata` | everything — students, groups, tests, answers, scores |
| `conative_uploads` | the images from uploaded test papers |

They survive `stop`, `start`, `restart`, and updating to a new version. The one
command that deletes them is `docker compose down -v`. Do not run that.

The `backups` folder *is* in the folder with the compose file, deliberately —
it is the one thing you can see, copy and carry away, and the only thing that
survives `down -v`.

Copying the compose file to another computer does **not** copy your data. The
backups are what moves it.

---

## Backups

Backups run on their own. There is nothing to remember and nothing to type.

A fourth container writes one **every night at 02:30**, and one **every time the
stack starts**, so there is always at least one backup from the current run.
They land in a `backups` folder next to your compose file:

```
conative/
  docker-compose.yml
  backups/
    2026-09-03_0230/
      db.dump          students, groups, tests, answers, scores
      uploads.tar.gz   the images from your test papers
    2026-09-04_0230/
      ...
```

The two files in a dated folder belong together: the database refers to the
images by name, so one without the other is half a backup. A folder only gets
its dated name once both files are finished, so the newest one is always a
complete, matching pair — there is no such thing as a half-written backup here.

The **last 7** are kept. Older ones are deleted automatically, so the folder
does not grow without limit.

Want one right now — before an update, or at the end of a test day?

```bash
docker compose restart backup
```

To check that backups are actually happening, `docker compose ps` reports
`backup` as **healthy** only while there is a backup less than 26 hours old. If
it ever reads unhealthy, backups have stopped and `docker compose logs backup`
will say why.

On Linux the files belong to `root`. You can read and copy them as yourself;
deleting one by hand needs `sudo`, which is why the pruning above is automatic.

### Copy them off this computer

**A backup that only exists on this laptop is not a backup.** The disk that
fails takes it with it, and that is the exact day you need it.

The simplest thing that keeps working without anyone remembering: put the whole
`conative` folder — compose file, `backups` and all — **inside a synced cloud
folder**, the one the Google Drive, Dropbox or OneDrive desktop app already
watches. Every night's backup is then uploaded on its own. Nobody has to do
anything, which is the only kind of routine that survives a busy term.

If you would rather not use cloud storage: keep a USB drive for this and drag
the newest dated folder onto it after every test day. Use two drives and
alternate, so a corrupt one is never the only copy.

## Restore from backup

On a fresh machine, or on this one after something went badly wrong. You need
Docker installed, `docker-compose.yml`, and one dated backup folder. Put the
backup folder inside a `backups` folder next to the compose file, as it was.

**1. Start the database on its own** and wait for it to be ready:

```bash
docker compose up -d --wait db
```

**2. Restore the database.** Use your own dated folder in place of this one:

```bash
docker compose exec -T db pg_restore -U conative -d conative --clean --if-exists \
  < backups/2026-09-04_0230/db.dump
```

**3. Restore the images:**

```bash
docker run -i --rm -v conative_uploads:/data alpine \
  sh -c 'rm -rf /data/* && tar xzf - -C /data' \
  < backups/2026-09-04_0230/uploads.tar.gz
```

**4. Start everything:**

```bash
docker compose up -d --wait
```

Then open <http://localhost:8080> and sign in. Everything comes back: students,
groups, papers, submitted answers and scores, the figures in the questions, and
the admin password you had set — that is in the backup too, so the one printed
further up this page will *not* work.

The database is started alone in step 1 on purpose. Bringing the whole stack up
first would have the backup container take a snapshot of the still-empty
database before you have restored anything.

---

## Updating

Download the current `docker-compose.yml` over your old one, then:

```bash
curl -O https://raw.githubusercontent.com/atharvrawal/conative-education/main/docker-compose.yml
docker compose pull
docker compose up -d
```

Any database changes the new version needs are applied automatically while it
starts. Your data is kept. Take a fresh backup first anyway — `docker compose
restart backup`, and check the new folder appeared before you pull.

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
- **`backup` says unhealthy.** No backup has been written in the last 26
  hours. `docker compose logs backup` gives the reason — nearly always no room
  left on the disk. `docker compose restart backup` retries immediately.
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
WEB_PORT=9000            # if something else is already using 8080
ADMIN_USERNAME=priya     # a different admin username
BACKUP_TIME=23:45        # when the nightly backup runs, 24-hour clock
BACKUP_KEEP=14           # how many nightly backups to keep
TZ=Asia/Kolkata          # the clock BACKUP_TIME is read against
```

Then `docker compose up -d`. Every value in the compose file can be set this
way.

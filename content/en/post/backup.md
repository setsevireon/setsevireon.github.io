---
title: Sysadmin's Home Backup
date: 2023-12-02
tags: [backup, restic, systemd]
author: Karl-Marx
description: Automated backup with restic, rclone and systemd
---

**Disclaimer:** this is not a tutorial.

**Another disclaimer:** I say *home* as in the place where I live, not as in
`/home/marx`. Although I suppose it works either way.

## Which software?

### Restic

The backup tool. I chose it first and invented a justification after,
so, no point in pretending there is a good motive. But that is philosophical
matter.

It does backup stuff, that is what's important.

### Rclone

Restic supports a handful of repository types, alas the one I need is not one
of them. Fortunately, restic allows us to use rclone as backend, which
supports mostly any cloud storage provider.

### Systemd

Sorry if you are on board of the "I hate systemd, it is the most disgusting
piece of software ever written" train. Use *cron* instead, it will work just
as well.

## Initializing repositories

First I created a rclone remote called `google` for Google Drive (no need to
judge me I already do it 🤷).

```shell
rclone config
```

Default options are fine, but encryption is a nice addition.

Initializing local repository:

```shell
restic init --repo /media/seagate/backup
```

This is just a big hard drive I have for local backup, for things that would be
cumbersome to lose but it wouldn't hurt.

Initializing remote repository:

```shell
restic init --repo rclone:google:backup
```

For data I cannot or will not afford to loose.

## My backup logic

### Level 0
If I loose these files I will have a panic attack. I am exaggerating, or perhaps
not. I hope to never find out.

### Lever 1
Mostly as important as the Level 0, but as it considerably more space-hungry
I am willing to pretend it is not that important. Don't say anything, it makes
sense in my head 🙉.

### Lever 2
This one exists to make my life easier in case I do something stupid.

### My backup strategy

Simple:
- l0 -> local and remote
- l1 -> local and remote
- l2 -> local

My intention is to run these backups hourly, that will make the snapshot
list unreasonably long. Unless, offcourse, i make up a rule to determine which
ones to keep.

I decided to keep
- 12 hourly
- 7 daily
- 4 weekly
- 12 monthly
- 5 yearly

## The backup command

Local 0:

```shell
restic backup --verbose \
  --group-by host,paths,tags \
  --tag l0,systemd \
  --password-file ~/.config/restic/password \
  --repo /media/seagate/backup \
  ~/Documents ~/Secrets ~/Logbook ~/Creations
```

Local 1:

```shell
restic backup --verbose \
  --group-by host,paths,tags \
  --tag l1,systemd \
  --password-file ~/.config/restic/password \
  --repo /media/seagate/backup \
  ~/Documents ~/Secrets ~/Logbook ~/Creations ~/Pictures ~/Templates ~/Sophie ~/Code
```

Local 2:

```shell
restic backup --verbose \
  --group-by host,paths,tags \
  --tag l2,systemd \
  --password-file ~/.config/restic/password \
  --repo /media/seagate/backup \
  ~/
```

Remote 0:

```shell
restic backup --verbose \
  --group-by host,paths,tags \
  --tag l0,systemd \
  --password-file ~/.config/restic/password \
  --repo rclone:google:backup \
  ~/Documents ~/Secrets ~/Logbook ~/Creations
```

Remote 1:

```shell
restic backup --verbose \
  --group-by host,paths,tags \
  --tag l1,systemd \
  --password-file ~/.config/restic/password \
  --repo rclone:google:backup \
  ~/Documents ~/Secrets ~/Logbook ~/Creations ~/Pictures ~/Templates ~/Sophie ~/Code
```

## The forget command

Local:

```shell
restic forget --verbose \
  --password-file ~/.config/restic/password \
  --repo /media/seagate/backup \
  --keep-hourly 12 \
  --keep-daily 7 \
  --keep-weekly 4 \
  --keep-monthly 12 \
  --keep-yearly 5 \
  --prune
```

Remote:

```shell
restic forget --verbose \
  --password-file ~/.config/restic/password \
  --repo rclone:google:backup \
  --keep-hourly 12 \
  --keep-daily 7 \
  --keep-weekly 4 \
  --keep-monthly 12 \
  --keep-yearly 5 \
  --prune
```


## Systemd unities

### Service `[ ~/.config/systemd/user/restic@.service ]`

```systemd
[Unit]
Description=Restic backup (%I)

[Service]
Type=oneshot

Environment="RESTIC_local0_TAGS=l0"
Environment="RESTIC_local0_REPOSITORY=/media/seagate/backup"
Environment="RESTIC_local0_TARGET=%h/Documents %h/Secrets %h/Logbook %h/Creations"

Environment="RESTIC_local1_TAGS=l1"
Environment="RESTIC_local1_REPOSITORY=/media/seagate/backup"
Environment="RESTIC_local1_TARGET=%h/Documents %h/Secrets %h/Logbook %h/Creations %h/Pictures %h/Templates %h/Sophie %h/Code"

Environment="RESTIC_local2_TAGS=l2"
Environment="RESTIC_local2_REPOSITORY=/media/seagate/backup"
Environment="RESTIC_local2_TARGET=%h"

Environment="RESTIC_google0_TAGS=l2"
Environment="RESTIC_google0_REPOSITORY=rclone:google:backup"
Environment="RESTIC_google0_TARGET=%h/Documents %h/Secrets %h/Logbook %h/Creations"

Environment="RESTIC_google1_TAGS=l1"
Environment="RESTIC_google1_REPOSITORY=rclone:google:backup"
Environment="RESTIC_google1_TARGET=%h/Documents %h/Secrets %h/Logbook %h/Creations %h/Pictures %h/Templates %h/Sophie %h/Code"

ExecStart=flock /tmp/restic-systemd.lock bash -c 'restic backup --verbose --group-by "host,paths,tags" --tag "${RESTIC_%i_TAGS},systemd" --password-file %h/.config/restic/password --repo ${RESTIC_%i_REPOSITORY} ${RESTIC_%i_TARGET}'
ExecStartPost=flock /tmp/restic-systemd.lock restic forget --verbose --password-file %h/.config/restic/password --repo ${RESTIC_%i_REPOSITORY} --keep-hourly 12 --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --keep-yearly 5 --prune

```

### Timer `[ ~/.config/systemd/user/restic@.timer ]`

```systemd
[Unit]
Description=Restic backup (%I)

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

## Enable the timer

```shell
systemctl --user daemon-reload
systemctl --user enable --now restic@local0.timer
systemctl --user enable --now restic@local1.timer
systemctl --user enable --now restic@local2.timer
systemctl --user enable --now restic@google0.timer
systemctl --user enable --now restic@google1.timer
```

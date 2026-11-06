# Privacy Policy - Vikunja for TerraMaster TOS 7

**Application**: Vikunja (`app_id: vikunja`)
**Publisher**: waskevin
**Policy version**: 1.0
**Effective date**: 2026-10-29
**Contact**: https://github.com/waskevin/vikunja/issues

This policy covers the TOS 7 application package distributed by waskevin
(the `vikunja.tar.gz` archive and the `config.ini` / `vikunja.lang` / `vikunja.svg` /
`docker-compose.yml` files inside it) and the container it starts on your
device.  It does not cover the Vikunja project's own website or any service you
connect the application to.

## What the package itself does

- It runs one Docker container from the public image `vikunja/vikunja:2.6`.
- It does not contain analytics, telemetry, advertising or tracking code.
- It does not send any data to waskevin or to any third party.
- It does not open network connections on its own; the only traffic is what
  the application and your browser make.

## Data the application stores

All application data (database, configuration, attachments) is written to
`/Volume<N>/DockerAppData/vikunja/data/` on your own device.  Nothing is
uploaded anywhere by the packaging.

## Data the application may send

- Nothing by default.  Vikunja talks only to its own database on your
  device.  If you configure an SMTP server for reminders, mails are
  sent through that server.

## Removing your data

Uninstalling the application from the TOS App Center stops and removes the
container.  Deleting the folder `DockerAppData/vikunja/` on your device removes
all remaining data; nothing is left elsewhere.

## Changes

Any change to this policy is published in this repository together with the
version of the package it applies to.

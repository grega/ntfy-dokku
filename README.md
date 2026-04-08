# ntfy-dokku

Minimal Dokku deployment wrapper for [ntfy](https://ntfy.sh), a self-hosted push notification service.

## Prerequisites

- A Dokku server with the [letsencrypt plugin](https://github.com/dokku/dokku-letsencrypt) installed
- A domain pointed at your Dokku server (e.g. `ntfy.your-domain.com`)
- The [ntfy iOS app](https://apps.apple.com/us/app/ntfy/id1625396347) or [Android app](https://play.google.com/store/apps/details?id=io.heckel.ntfy)

## Deploy

On your Dokku server, create the app and configure storage:

```bash
dokku apps:create ntfy
dokku domains:set ntfy ntfy.your-domain.com
dokku ports:set ntfy http:80:80
dokku storage:ensure-directory ntfy-data
dokku storage:mount ntfy /var/lib/dokku/data/storage/ntfy-data:/var/cache/ntfy
dokku letsencrypt:enable ntfy
```

Then push this repo to Dokku:

```bash
git remote add dokku dokku@your-server:ntfy
git push dokku main
```

## Connect your phone

1. Open the ntfy app on your phone
2. Go to Settings and add your server URL (`https://ntfy.your-domain.com`)
3. Set it as the default server
4. Subscribe to a topic (e.g. `aircraft-alerts`)

## Test

```bash
curl -d "Hello from Dokku!" https://ntfy.your-domain.com/aircraft-alerts
```

## Optional: Access control

By default ntfy allows anyone to publish and subscribe to topics. To restrict access:

```bash
dokku storage:ensure-directory ntfy-config
dokku storage:mount ntfy /var/lib/dokku/data/storage/ntfy-config:/etc/ntfy
```

Create `/var/lib/dokku/data/storage/ntfy-config/server.yml` on the host:

```yaml
auth-default-access: "deny-all"
auth-file: "/var/cache/ntfy/user.db"
```

Then rebuild and add a user:

```bash
dokku ps:rebuild ntfy
dokku run ntfy ntfy user add --role=admin your-username
dokku run ntfy ntfy token add your-username
```

Use the returned token for authenticated access.

## Updating ntfy

To update to a newer version, change the tag in the `Dockerfile` and push again:

```bash
git push dokku main
```

# POC Restic with Docker

Resources:

- [Restic](https://restic.readthedocs.io/)
- [Restic Backup Docker Container](https://github.com/Lobaro/restic-backup-docker)

Note: if you need to backup PostgreSQL database, see [`poc-postgresql-walg`](https://github.com/stephane-klein/poc-postgresql-walg)

## Backup scenario

Put some data to backup in `data/`:

```
$ mkdir -p data
$ (cd data; curl -L https://github.com/restic/restic/archive/master.tar.gz | tar -xz)
```

Start Minio (s3 like) service:

```
$ docker-compose up -d s3
```

Start Restic service:

```
$ docker-compose up -d restic
$ docker-compose logs -f restic
```

Default backup cron configuration: `BACKUP_CRON="0 */6 * * *"`

This is how to execute backup now:

```
$ docker-compose exec restic /bin/backup.sh
Starting Backup at 2020-04-04 09:10:59
Backup Successfull
Finished Backup at 2020-04-04 09:11:01 after 2 seconds
```

You can go to http://127.0.0.1:9000/minio/bucket1/ to see backup data in Minio (S3 like) bucket.

## Restoration scenario

Now I would like restore file outside Docker.

```
$ brew install restic
```

```
$ source source.env
```

```
$ restic -r s3:http://127.0.0.1:9000/bucket1 snapshots
enter password for repository: secret
repository bf543bcb opened successfully, password is correct
created new cache in /Users/stephane/Library/Caches/restic
ID        Time                 Host          Tags        Paths
--------------------------------------------------------------
06d0afe9  2020-04-04 11:05:40  9aef423f0463              /data
a90a2c8b  2020-04-04 11:10:59  9aef423f0463              /data
--------------------------------------------------------------
2 snapshots
```

```
$ restic -r s3:http://127.0.0.1:9000/bucket1 check
using temporary cache in /var/folders/91/5mgrtw6n4w1c9rr92q4sqrb40000gn/T/restic-check-cache-300660334
enter password for repository: secret
repository bf543bcb opened successfully, password is correct
created new cache in /var/folders/91/5mgrtw6n4w1c9rr92q4sqrb40000gn/T/restic-check-cache-300660334
create exclusive lock for repository
load indexes
check all packs
check snapshots, trees and blobs
no errors were found
```

Restore data to `data2/`:


```
$ restic -r s3:http://127.0.0.1:9000/bucket1 restore a90a2c8b --target ./data2/
enter password for repository:
repository bf543bcb opened successfully, password is correct
restoring <Snapshot a90a2c8b of [/data] at 2020-04-04 09:10:59.0699908 +0000 UTC by @9aef423f0463> to ./data2/
```

```
$ du -h -d0 data
26M	data
$ du -h -d0 data2
26M	data2
```

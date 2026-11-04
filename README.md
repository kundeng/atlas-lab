# atlas-lab

Docker images for an Apache Atlas 2.5.0 lab: Atlas with the PostgreSQL graph backend, Solr 8 and ZooKeeper 3.9. Each image is published for `linux/amd64` and `linux/arm64`, so it runs natively on Intel machines and on Apple Silicon. Images only: no data.

Apache publishes no Atlas images, and the community images on Docker Hub stop at Atlas 2.3 with the embedded HBase store and amd64 only. These are built from the Atlas 2.5.0 release tag with its own `dev-support/atlas-docker` Dockerfiles; the Maven build was run once and the server tarball reused for both architectures.

| Image | Based on |
| --- | --- |
| `ghcr.io/kundeng/atlas-lab/atlas:2.5.0` | Ubuntu 20.04, Java 8, `apache-atlas-2.5.0-server.tar.gz` |
| `ghcr.io/kundeng/atlas-lab/atlas-db:2.5.0` | `postgres:13.21` with Atlas's init script |
| `ghcr.io/kundeng/atlas-lab/atlas-solr:2.5.0` | `solr:8` with Atlas's collection config |
| `ghcr.io/kundeng/atlas-lab/atlas-zk:2.5.0` | `zookeeper:3.9.2` |

## Run it

```bash
docker compose up -d
curl -u admin:atlasR0cks! http://127.0.0.1:21000/api/atlas/admin/version   # {"Version":"2.5.0", ...}
```

The UI is at http://127.0.0.1:21000 (the newer interface at `/n3/index.html`). The first start takes a few minutes while Atlas creates its schema. Kafka is left out: Atlas runs without it when no hooks send notifications.

`admin` / `atlasR0cks!` and the database password are the Apache dev-support defaults. Use these images for a local lab only, with the ports bound to 127.0.0.1 as the compose file does.

Set `ATLAS_PORT` to change the host port. `docker compose down -v` deletes the data.

## Restoring a data snapshot

The four volumes (`atlas-lab_db`, `atlas-lab_solr`, `atlas-lab_zk-data`, `atlas-lab_zk-datalog`) can be filled from a tarball holding `db/`, `solr/`, `zk-data/` and `zk-datalog/`, taken while the stack was stopped:

```bash
docker compose create
docker run --rm --entrypoint tar \
  -v atlas-lab_db:/r/db -v atlas-lab_solr:/r/solr -v atlas-lab_zk-data:/r/zk-data -v atlas-lab_zk-datalog:/r/zk-datalog \
  -v "$PWD":/in -w /r ghcr.io/kundeng/atlas-lab/atlas-db:2.5.0 --numeric-owner -xzf /in/atlas-data.tar.gz
docker compose up -d
```

A snapshot taken on amd64 restored and ran on arm64 (PostgreSQL 13, Solr 8).

## License

Apache Atlas, its Dockerfiles and scripts are Apache License 2.0, from https://github.com/apache/atlas at tag `release-2.5.0`.

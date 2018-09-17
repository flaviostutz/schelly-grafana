# schelly-grafana
Grafana pre installed with custom plugins and dashboards for Schelly metrics

For more details on Schelly, visit https://github.com/flaviostutz/schelly

## Usage (full example)

docker-compose.yml

```
version: '3.5'

services:

  schelly-grafana:
    image: flaviostutz/schelly-grafana
    ports:
      - 3000:3000
    volumes:
      - grafana:/data

  schelly:
    image: flaviostutz/schelly
    ports:
      - 8080:8080
    environment:
      - LOG_LEVEL=debug
      - BACKUP_NAME=test
      - WEBHOOK_URL=http://schelly-restic:7070/backups
      - BACKUP_CRON_STRING=0/10 * * * * *
      - WEBHOOK_HEADERS=k1=v1,k2=v2
      # - WEBHOOK_CREATE_BODY=""
      # - WEBHOOK_DELETE_BODY=""
      - WEBHOOK_GRACE_TIME=30
      - RETENTION_MINUTELY=10
      # - RETENTION_HOURLY=""
      # - RETENTION_DAILY=""
      # - RETENTION_WEEKLY=""
      # - RETENTION_MONTHLY=""
    volumes:
      - schelly:/var/lib/schelly/data

  schelly-restic:
    image: flaviostutz/schelly-restic
    ports:
      - 7070:7070
    environment:
      - RESTIC_PASSWORD=123
      - LOG_LEVEL=debug
      - PRE_BACKUP_COMMAND=dd if=/dev/zero of=/backup-source/TESTFILE bs=100MB count=2
      - POST_BACKUP_COMMAND=rm /backup-source/TESTFILE
      - SOURCE_DATA_PATH=/backup-source/TESTFILE
      - TARGET_DATA_PATH=/backup-repo
    volumes:
      - restic-backup-source:/backup-source
      - restic-backup-repo:/backup-repo

  prometheus:
    image: flaviostutz/prometheus
    ports:
      - 9090:9090
    environment:
      - STATIC_SCRAPE_TARGETS=schelly@schelly:8080
    volumes:
      - prometheus:/prometheus
          
networks:
  default:
    ipam:
      driver: default
      config:
        - subnet: 192.168.15.0/24

volumes:
  grafana:
  prometheus:
  schelly:
  restic-backup-source:
  restic-backup-repo:

```

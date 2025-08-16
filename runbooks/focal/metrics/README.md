# Metrics

## Usage

Start InfluxDB and Grafana.

    sudo docker compose up -d

Setup the Grafana datasources and dashboards.

    python3 bootstrap.py

Log in to http://localhost:3000. Set the Telegraf datasource password.

## Hacking

Start an InfluxDB shell.

    sudo docker exec -it metrics_influxdb_1 /bin/bash
    influx -username admin -password admin

Dump Grafana datasources.

    curl -s "http://admin:admin@localhost:3000/api/datasources/" -H "Content-Type: application/json" | python3 -m json.tool

# Grafana

Grafana with Prometheus and InfluxDB


1. Pull all the packages
docker pull prom/prometheus:latest
docker pull prom/node-exporter:latest
docker pull grafana/grafana:latest
docker pull influxdb:latest
Configure Prometheus

2. mkdir -p /etc/prometheus
vi prometheus.yml # Paste below Lines in prometheus.yml
# scrape configuration scraping a Node Exporter and the Prometheus # # # #server itself
scrape_configs:
# Scrape Prometheus itself every 5 seconds.
- job_name: ‘prometheus’
scrape_interval: 5s
static_configs:
- targets: [‘10.x.x.x:9090’]

3. Run promrtheus
docker run -d -p 9090:9090 --name prometheus -v /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus:latest --config.file=/etc/prometheus/prometheus.yml

4. Run Grafana
docker run -d -p 3000:3000 --name grafana -e “GF_SECURITY_ADMIN_PASSWORD=admin_password” -v ~/grafana_db:/var/lib/grafana grafana/grafana:latest

5. Run influxdb (optional): If want to store data
docker run -d --name=influxdb --restart on-failure -p 8086:8086 -v influxdb_data:/var/lib/influxdb influxdb:latest -config /etc/influxdb/influxdb.conf

Now let them provide the permission to access DB
$docker exec -it influxdb bash
$influx
$CREATE DATABASE prometheus
$use prometheus
$CREATE RETENTION POLICY “30_days” ON “prometheus” DURATION 30d REPLICATION 1 DEFAULT
$show RETENTION POLICIES
To enable read/write in remote influxdb (add below configuration parameters)
remote_write:
- url: “http://10.x.x.x:8086/api/v1/prom/write?db=prometheus"
remote_read:
- url: “http://10.x.x.x:8086/api/v1/prom/read?db=prometheus"

6. Run node-exporter on evrey node to monitor
docker run -d -p 9100:9100 --name node-exporter -v “/proc:/host/proc” -v “/sys:/host/sys” -v “/:/rootfs” --net=”host” prom/node-exporter:latest --path.procfs /host/proc --path.sysfs /host/proc --collector.filesystem.ignored-mount-points “^/(sys|proc|dev|host|etc )($|/)”
To Add nodes to prometheus (add below configuration parameters)
vi prometheus.yml # Copy line of code given below
# Scrape the Node Exporter every 5 seconds.
- job_name: ‘node’
scrape_interval: 5s
static_configs:
#Add target which u want to Monitor
- targets: [‘10.x.x.x:9100’, ‘10.x.x.x:9100’, ‘10.x.x.x:9100’, ‘10.x.x.x:9100’, ‘10.x.x.x:9100’]
For docker-compose and correct formatting please follow the below link
https://gitlab.com/vineetkumar03/grafana
To check Prometheus target : http://<IP>:9090
Now to set up grafana:
a. Login to grafana and go to “create a datasource ” link and then choose prometheus -> fill prometheus URL and port (other fill bydefault value)and save it.
b. Go to + -> choose import and fill dashboard id or JSON file take it from below link
c. Go to https://grafana.com/grafana/dashboards/ and choose any dashboard type , i choose two https://grafana.com/grafana/dashboards/405 (id will be 405)and https://grafana.com/grafana/dashboards/1860 (id will be 1860)after that save that dashboard


### Prerequisites
- [Docker](https://www.docker.com/)

### Run monitoring service
- Run `com.evolutiongaming.bootcamp.monitoring.Main`

### Set up Prometheus stack

```bash
git clone https://github.com/vegasbrianc/prometheus.git
```

- Follow [the instructions](https://github.com/vegasbrianc/prometheus/blob/master/README.md) to run it.

- You should see the following in your console:
```bash
Creating network prom_monitor-net
Creating service prom_cadvisor
Creating service prom_grafana
Creating service prom_prometheus
Creating service prom_node-exporter
Creating service prom_alertmanager
```

- Check that all of the services are up:
```bash
docker ps

CONTAINER ID        IMAGE                       COMMAND                  CREATED              STATUS              PORTS               NAMES
093d9729cb2b        prom/alertmanager:latest    "/bin/alertmanager -…"   About a minute ago   Up About a minute   9093/tcp            prom_alertmanager.1.d0ebj8k3zq2dlhgny2rzb9idx
4be79d2eb882        prom/prometheus:latest      "/bin/prometheus --c…"   About a minute ago   Up About a minute   9090/tcp            prom_prometheus.1.e6qvyv2jl57pum4fba3sjrnz2
ce185659ec76        prom/node-exporter:latest   "/bin/node_exporter …"   About a minute ago   Up About a minute   9100/tcp            prom_node-exporter.m78vyn14ro2nt1c8tn4h2hoyz.nccv13q6ia03dv58qnnnjvasy
7f8ae1294021        grafana/grafana:latest      "/run.sh"                About a minute ago   Up About a minute   3000/tcp            prom_grafana.1.yneb73zah7dnfoc7g8qrv1oqa
681e0eb261fe        google/cadvisor:latest      "/usr/bin/cadvisor -…"   2 minutes ago        Up 2 minutes        8080/tcp            prom_cadvisor.m78vyn14ro2nt1c8tn4h2hoyz.8agtc6zkmv2zanv3yx81uzomp
```

### Configuring Prometheus data store
#### macOS
- Open `prometheus/prometheus.yml`
- Add the following into the `scrape_configs` section:
```yaml
- job_name: 'main-monitoring'
    scrape_interval: 5s
    static_configs:
         - targets: ['host.docker.internal:9000']
```

- Redeploy docker stack.
```bash
docker stack rm prom
HOSTNAME=$(hostname) docker stack deploy -c docker-stack.yml prom
```

- Observe metrics in [your app's endpoint](http://localhost:9000/metrics).

- Observe metrics in [Prometheus](http://prometheus.localhost:9090/). 

- Go to [grafana.localhost:3000](http://grafana.localhost:3000) and login to Grafana `admin/foobar`.

- [Import JVM dashboard](http://localhost:3000/dashboard/import) from [`jvm.json`](jvm.json).

- Observe JVM metrics in Grafana.

- Add a new panel in Grafana of `rate(requests{instance="host.docker.internal:9000",job="main-monitoring"}[1m])`, observe this metric.

- Run `ab -n 100 -c 10 http://127.0.0.1:9000/normal-distribution-delay/5000/1000` to generate load, observe metrics.

- Try stopping the `Main` app and see how the memory pools drop to 0.

#### Troubleshooting
Q: Dashboard is imported, but it shows "No data points"
A: Adjust time window to smaller period of time. By default it show 90 days.

Q: Windows has been adjusted, still see nothing.
A: Open [http://localhost:9090/targets](http://localhost:9090/targets), check `main-monitoring` is in `UP` state. 
# bootcamp_mon

Для установки Grafana, Prometheus and Node Exporter выполните следующие шаги:
1. Deploy stack
```
git clone https://github.com/dzift/bootcamp_mon.git

sudo docker swarm init

sudo docker stack deploy -c bootcamp_mon/docker-compose.yml monitoring
```
2. Добавьте prometheus как источник данных по адресу `http://prometheus:9090` для grafana.
3. Добавьте новый источник метрик для prometheus, добавив в файл `/var/lib/docker/volumes/monitoring_prom-configs/_data/prometheus.yml` дополнительные параметры` scrape_configs:`:

```
  - job_name: 'node-exporter'

    static_configs:
      - targets: ['node-exporter:9100']
```

4. Перезапустите prometheus командой:

```
sudo docker ps | grep prometheus | awk '{print $1}' | xargs docker kill -s SIGHUP
```

5. Импортируйте новый дашборд для grafana по ссылке: https://grafana.com/grafana/dashboards/1860.

6. Добавьте репозиторий в диспетчер пакетов:

```
cat <<EOF | sudo tee /etc/apt/sources.list.d/influxdata.list
deb https://repos.influxdata.com/ubuntu $(lsb_release -cs) stable
EOF

```
Добавьте ключ:

```
sudo curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -

```
Установите TELEGRAF:

```
sudo apt update
sudo apt install telegraf

```

Запустите и включите службу для запуска при загрузке:
```
sudo systemctl enable --now telegraf
sudo systemctl is-enabled telegraf
sudo systemctl status telegraf
```

7.  Настройте Telegraf для работы с Prometheus, раскомментировав в конфигурацию параметры :

```
[[outputs.prometheus_client]]
  ## Address to listen on.
  listen = ":9273"
```

8. Добавьте Telegraf в конфигурацию Prometheus:

```
scrape_configs:
- job_name: "telegraf"
  static_configs:
  - targets:
    - "172.17.0.1:9273"
    
```

9. Перезапустите Prometheus командой:
```
sudo docker ps | grep prometheus | awk '{print $1}' | xargs docker kill -s SIGHUP
```

10. Перезапустите Telegraf командой:

```
sudo systemctl status telegraf
```

11. Для настройки мониторинга по healthcheck в кофнигурации Telegraf настройте pluguin http_response:

```
[[inputs.http_response]]
  ## List of urls to query.
  urls = ["http://localhost:8080/status"]
```
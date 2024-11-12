# Установка и настройка Grafana + Prometheus + Node Exporter + VictoriaMetrics на Kali Linux (ARM)
## Этот документ описывает процесс установки и настройки Grafana, Prometheus, Node Exporter и VictoriaMetrics на Kali Linux, работающем в виртуальной машине Parallels Desktop на Mac M1 ARM.
### 1. Установка Prometheus
#### 1. Загрузка версии Prometheus для ARM:

```
sudo apt update
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-arm64.tar.gz
```
#### 2. Распаковка и установка:

```
tar xvfz prometheus-2.45.0.linux-arm64.tar.gz
cd prometheus-2.45.0.linux-arm64
sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/
sudo useradd --no-create-home --shell /bin/false prometheus
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```


#### 3. Создание службы Prometheus:
```
sudo nano /etc/systemd/system/prometheus.service
```


##### Добавьте следующий контент:
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

#### 4. Запуск и активация службы:
```
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```


### 2. Установка Node Exporter
#### 1. Загрузка версии для ARM:
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-arm64.tar.gz
tar xvfz node_exporter-1.6.0.linux-arm64.tar.gz
sudo mv node_exporter-1.6.0.linux-arm64/node_exporter /usr/local/bin/
```

#### 2. Создание пользователя и конфигурации службы:
```
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```
#### Создайте файл службы:
```
sudo nano /etc/systemd/system/node_exporter.service
```

#### Поместите файл:
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

#### 3. Запуск Node Exporter:
```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```


### 3. Установка VictoriaMetrics:
#### 1. Загрузка версии для ARM:
```
wget https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.90.0/victoria-metrics-linux-arm64-v1.90.0.tar.gz
tar -xvf victoria-metrics-linux-arm64-v1.90.0.tar.gz
sudo mv victoria-metrics-prod /usr/local/bin/victoria-metrics
```

#### 2. Запуск VictoriaMetrics:
```
nohup victoria-metrics -storageDataPath=/var/lib/victoria-metrics &
```

### 4. Установка Grafana:
#### 1. Установите Docker, если он еще не установлен:
```
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

#### 2. Запустите Grafana в контейнере Docker:
```
sudo docker run -d -p 3000:3000 --name=grafana grafana/grafana-oss:latest
```

#### 3. Проверьте, что контейнер работает:
```
sudo docker ps
```

### 5. Конфигурация Docker-compose.yaml ( ко всему тому, что уже создалось ) 
```
  node-exporter:
    image: prom/node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    container_name: exporter
    hostname: exporter
    command:
      - --path.procfs=/host/proc
      - --path.sysfs=/host/sys
      - --collector.filesystem.ignored-mount-points
      - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)
    ports:
      - 9100:9100
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default
  vmagent:
    container_name: vmagent
    image: victoriametrics/vmagent:v1.105.0
    depends_on:
      - "victoriametrics"
    ports:
      - 8429:8429
    volumes:
      - vmagentdata:/vmagentdata
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - "--promscrape.config=/etc/prometheus/prometheus.yml"
      - "--remoteWrite.url=http://victoriametrics:8428/api/v1/write"
    restart: always
  victoriametrics:
    container_name: victoriametrics
    image: victoriametrics/victoria-metrics:v1.105.0
    ports:
      - 8428:8428
      - 8089:8089
      - 8089:8089/udp
      - 2003:2003
      - 2003:2003/udp
      - 4242:4242
    volumes:
      - vmdata:/storage
    command:
      - "--storageDataPath=/storage"
      - "--graphiteListenAddr=:2003"
      - "--opentsdbListenAddr=:4242"
      - "--httpListenAddr=:8428"
      - "--influxListenAddr=:8089"
      - "--vmalert.proxyURL=http://vmalert:8880"
    restart: always
```


### 6. Настройка визуализации в Grafana

#### 1. Заходим на порт, зарезервированный нашей графаной (localhost:3000)

#### 2. Добавляем Data Source --> Prometheus 

#### 3. Добавляем DashBoard --> Add Visualisation --> Import DashBoard --> 1860


### 7. ПРИЛОЖЕНИЕ (Необходимые скрины к проекту) 
#### 1. Демонстрация работоспособности графаны

<img width="1436" alt="image" src="https://github.com/user-attachments/assets/eaa616ee-42e1-47b9-853c-7ffbc74e402b">

#### 2. Демонстрация docker-compose 

<img width="1299" alt="image" src="https://github.com/user-attachments/assets/a5776843-24d7-456c-813b-73e132b6ab1d">

### 3. Демонстрация Prometheus

<img width="1440" alt="image" src="https://github.com/user-attachments/assets/520139bf-3f36-49e1-8360-6dbb926f0bb9">


### 4. Демонстрация иерархии файлов

<img width="569" alt="image" src="https://github.com/user-attachments/assets/61627314-edd3-4173-be5e-60087e1653f4">



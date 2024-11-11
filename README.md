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

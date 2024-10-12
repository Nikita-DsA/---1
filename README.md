# Установка системы мониторинга для OS Kali Linux
## Docker (Grafana + Prometheus + Node Exporter)
### Этап 1: Обновление пакетов системы: 

```
sudo apt update 
```
### Этап 2: Установка Docker 

```
sudo apt-install -y docker.io
```
```
sudo systemctl enable docker --now
```

### Этап 3: Установка Grafana (ARM64)
```
wget https://dl.grafana.com/oss/release/grafana_8.2.2_arm64.deb 
```
```
sudo dpkg -i grafana_8.2.2_amd64.deb
```

### 3.1 Запуск Grafana
```
sudo systemctl start grafana-server
```

### 3.2 Остановка служб Grafana
```
sudo systemctl stop grafana-server
```

### Этап 4: Установка Prometheus (ARM64)


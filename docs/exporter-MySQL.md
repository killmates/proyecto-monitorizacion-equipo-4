# MySQL Exporter para Prometheus

## ¿Qué hace el MySQL Exporter?

Es un exporter encargado de recopilar métricas internas de MySQL y exponerlas para que Prometheus pueda recogerlas.

## Instalación

### 1. Instalar MySQL

```bash
sudo apt install mysql-server -y
sudo systemctl enable --now mysql
```

### 2. Crear usuario MySQL para el exporter

Entramos en MySQL:

```bash
sudo mysql
```

Ejecutamos los siguientes comandos:

```sql
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'ContraseñaSegura!' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 3. Descargar e instalar el MySQL Exporter

```bash
sudo useradd --no-create-home --shell /bin/false mysqld_exporter
cd /tmp
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.15.1/mysqld_exporter-0.15.1.linux-amd64.tar.gz
tar xvf mysqld_exporter-0.15.1.linux-amd64.tar.gz
sudo cp mysqld_exporter-0.15.1.linux-amd64/mysqld_exporter /usr/local/bin/
sudo chown mysqld_exporter:mysqld_exporter /usr/local/bin/mysqld_exporter
```

### 4. Crear el fichero de credenciales

```bash
sudo nano /etc/.mysqld_exporter.cnf
```

Contenido del fichero:

```ini
[client]
user=exporter
password=ContraseñaSegura!
host=localhost
```

Ajustar permisos:

```bash
sudo chown root:mysqld_exporter /etc/.mysqld_exporter.cnf
sudo chmod 640 /etc/.mysqld_exporter.cnf
```

### 5. Crear el servicio systemd

```bash
sudo nano /etc/systemd/system/mysqld_exporter.service
```

Contenido del fichero:

```ini
[Unit]
Description=MySQL Exporter
After=network-online.target

[Service]
User=mysqld_exporter
Group=mysqld_exporter
ExecStart=/usr/local/bin/mysqld_exporter \
  --config.my-cnf=/etc/.mysqld_exporter.cnf \
  --web.listen-address=0.0.0.0:9104

[Install]
WantedBy=multi-user.target
```

Iniciar el servicio:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now mysqld_exporter
```

### 6. Añadir el target en Prometheus

Editar `/etc/prometheus/prometheus.yml` y añadir el job de MySQL:

```yaml
scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['localhost:9104']
```

Reiniciar Prometheus:

```bash
sudo systemctl restart prometheus
```

## Verificación

Comprobar que el exporter está recogiendo métricas:

```bash
curl http://localhost:9104/metrics | grep mysql_up
```

Debe devolver:

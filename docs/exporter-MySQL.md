# MySQL Exporter

## ¿Qué hace el MySQL Exporter?

Es un exporter que recoge métricas de MySQL para que Prometheus pueda monitorizarlas.

## Instalación

### 1. Instalar MySQL

Primero instalamos MySQL en la máquina:

```bash
sudo apt install mysql-server -y
sudo systemctl enable --now mysql
```

### 2. Crear usuario para el exporter en MySQL

Entramos en MySQL:

```bash
sudo mysql
```

Dentro creamos el usuario con los permisos necesarios:

```sql
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'ContraseñaSegura!' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 3. Descargar el MySQL Exporter

Creamos el usuario del sistema y descargamos el binario:

```bash
sudo useradd --no-create-home --shell /bin/false mysqld_exporter
cd /tmp
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.15.1/mysqld_exporter-0.15.1.linux-amd64.tar.gz
tar xvf mysqld_exporter-0.15.1.linux-amd64.tar.gz
sudo cp mysqld_exporter-0.15.1.linux-amd64/mysqld_exporter /usr/local/bin/
sudo chown mysqld_exporter:mysqld_exporter /usr/local/bin/mysqld_exporter
```

### 4. Fichero de credenciales

Creamos el fichero donde guardamos el usuario y contraseña de MySQL:

```bash
sudo nano /etc/.mysqld_exporter.cnf
```

Dentro ponemos esto:

```ini
[client]
user=exporter
password=ContraseñaSegura!
host=localhost
```

Le damos los permisos correctos:

```bash
sudo chown root:mysqld_exporter /etc/.mysqld_exporter.cnf
sudo chmod 640 /etc/.mysqld_exporter.cnf
```

### 5. Crear el servicio

```bash
sudo nano /etc/systemd/system/mysqld_exporter.service
```

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

Arrancamos el servicio:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now mysqld_exporter
```

### 6. Añadir el target en Prometheus

Editamos el fichero de configuración de Prometheus:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Añadimos el job de MySQL:

```yaml
scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['localhost:9104']
```

Reiniciamos Prometheus para que coja los cambios:

```bash
sudo systemctl restart prometheus
```

## Comprobación

Para ver si funciona ejecutamos:

```bash
curl http://localhost:9104/metrics | grep mysql_up
```

Si devuelve `mysql_up 1` es que todo está bien. También se puede comprobar desde la web de Prometheus en `http://TU_IP:9090` buscando `mysql_up`.

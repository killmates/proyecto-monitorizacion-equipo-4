# MYSQL EXPORTER

## ¿Qué es?

Es una herramienta que se conecta a MySQL y saca métricas para que Prometheus las pueda ver.

## Instalación

1. Instalar MySQL

sudo apt install mysql-server -y
sudo systemctl enable --now mysql

2. Crear el usuario en MySQL

Entramos a MySQL:

sudo mysql

Creamos el usuario que va a usar el exporter para conectarse:

CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'ContraseñaSegura!' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
FLUSH PRIVILEGES;
EXIT;

3. Descargar el exporter

sudo useradd --no-create-home --shell /bin/false mysqld_exporter
cd /tmp
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.15.1/mysqld_exporter-0.15.1.linux-amd64.tar.gz
tar xvf mysqld_exporter-0.15.1.linux-amd64.tar.gz
sudo cp mysqld_exporter-0.15.1.linux-amd64/mysqld_exporter /usr/local/bin/
sudo chown mysqld_exporter:mysqld_exporter /usr/local/bin/mysqld_exporter

4. Fichero con la contraseña

Creamos un fichero donde guardamos el usuario y la contraseña que creamos antes:

sudo nano /etc/.mysqld_exporter.cnf

Ponemos esto dentro:

[client]
user=exporter
password=ContraseñaSegura!
host=localhost

Guardamos y ajustamos los permisos:

sudo chown root:mysqld_exporter /etc/.mysqld_exporter.cnf
sudo chmod 640 /etc/.mysqld_exporter.cnf

5. Crear el servicio para que arranque solo

sudo nano /etc/systemd/system/mysqld_exporter.service

Pegamos esto:

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

Arrancamos el servicio:

sudo systemctl daemon-reload
sudo systemctl enable --now mysqld_exporter

6. Decirle a Prometheus que recoja las métricas de MySQL

sudo nano /etc/prometheus/prometheus.yml

Añadimos esto:

scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['localhost:9104']

Reiniciamos para que coja los cambios:

sudo systemctl restart prometheus

## Comprobación

Ejecutamos esto para ver si funciona:

curl http://localhost:9104/metrics | grep mysql_up

Si sale mysql_up 1 es que está funcionando bien. También se puede entrar a Prometheus desde el navegador en http://TU_IP:9090 y buscar mysql_up.

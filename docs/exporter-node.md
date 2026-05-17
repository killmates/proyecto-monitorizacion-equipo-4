# NODE EXPORTER

## NOMBRE
Soufian Soulaimani

---

## QUE HACE EL EXPORTER
Node Exporter es un exporter oficial de Prometheus que se encarga de recoger metricas del sistema operativo y del hardware del servidor
(CPU, RAM, DISCO, RED) y exponerlas en formato HTTP para que Prometheus las pueda recoger.

--

## Como se instala

## 1. Crea usuario
sudo useradd --no-create-home --shell /bin/false node_exporter

## 2. Descargar Node Exporter
cd /tmp
wget
https://github.com/prometheus/node_exporter/releases/lastest/donwload/node_exporter-1.8.2.linux-amd64.tar.gz

## 3. Descomprimir
tar xvf node_exporter-1.8.2.linux-amd64.tar.gz

## 4. Mover binario
sudo cp node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/

## 5. Permisos
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
sudo chmod +x /usr/local/bin/node_exporter

## 6. Crear Servicio
sudo nano /etc/systemd/system/node_exporter.service

Contenido:

[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_eporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target

##7. Activar Servicio
sudo systemctl daemon-reload
sudo systemctl enable -now node_exporter

--

## Configuracion en Prometheus

Editar el archivo:

sudo nano /etc/prometheus/prometheus.yml

Aniadir:

scrape_configs:
 - job_name: 'node_exporter'
   static_configs:
    - targets: ['localhost:9100']

Reiniciar:

sudo systemctl restart prometheus

--

## 3. Metricas Importantes

##. 1. node_cpu_seconds_total
- tipo: Counter
- Que mide: Tiempo total de uso de l CPU desde que inicio el sistema
- Importancia: Permite calcular el uso del CPU con funciones como rate().

## 2. node_memory_MemTotal_bytes
- Tipo: Gauge
- Que mide: Memoria Disponible real para procesos
- Importancia: Es mas util que la memoria libre por que tiene cache y buffers

--

## Alerta propuesta

Si la memoria disponible baja del 10% del total durante mas de 5 minutos, se debe generar
una alerta para evitar fallos para la memoria.

--

## Limitacion

Node Exporter solo monitoriza el sistema operativo y hardware. No recoge metricas de 
aplicaciones, contenedores o servicios especificos.

## Verificacion

- Servicio activo:
  sudo systemctl status node_exporter

- Metricas
  http://localhost:9100/metrics

- Prometheus:
  http://localhost:9090 --> Status --> Targets (UP)

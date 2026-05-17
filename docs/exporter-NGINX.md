# ==> NGINX Exporter <==

**¿Que hace este Exporter?**
Extrae las métricas de rendimiento de un servidor NGINX y las traduce al formato compatible con Prometheus para que puedas monitorear tu servidor en tiempo real.

## Pasos a seguir para la istalación y configuración
**Ejecutamos en la terminal el siguiente comando para empezar con la instalación:** sudo apt update | sudo apt install nginx -y | sudo systemctl status nginx

**Ahora editamos el siguiente fichero:** sudo nano /etc/nginx/sites-available/default y añadimos las siguientes lineas...
```
location /nginx_status {
   stub_status on;
    allow 127.0.0.1;    * Solo permite que el exporter lea estos datos
    deny all;           * Prohíbe el acceso al resto del mundo
}
```
**Salvamos el fichero, aplicamos la configuracióny reiniciamos el NGINX:** sudo nginx -t && sudo systemctl restart nginx

**Ahora instalamos el Exporter con el siguiente comando:** wget https://github.com/nginxinc/nginx-prometheus-exporter/releases/download/v1.1.0/nginx-prometheus-exporter_1.1.0_linux_amd64.tar.gz

**Descomprimimos y lo movemos a una ubicación de ejecución:** tar -xvf nginx-prometheus-exporter_1.1.0_linux_amd64.tar.gz && sudo mv nginx-prometheus-exporter /usr/local/bin/

**Aeguramos el procesocon un usuario dedicado únicamente:** sudo useradd --no-create-home --shell /bin/false nginx_exporter

**Creamos el servicio automático editando el fichero siguiente:** sudo nano /etc/systemd/system/nginx_exporter.service

**Ahora añadimos estas lineas y salvamos:**
```
[Unit]
Description=Nginx Prometheus Exporter
After=network.target

[Service]
User=nginx_exporter
Group=nginx_exporter
Type=simple
ExecStart=/usr/local/bin/nginx-prometheus-exporter \
    -nginx.scrape-uri=http://127.0.0.1/nginx_status

[Install]
WantedBy=multi-user.target
Activamos el servicio
sudo systemctl daemon-reload
sudo systemctl enable nginx_exporter
sudo systemctl start nginx_exporter
Conectamos Prometheus con el Exporter editando lo siguiente
sudo nano /etc/prometheus/prometheus.yml
Añadimos estas líneas y salvamos
- job_name: 'nginx_metrics'
    static_configs:
      - targets: ['localhost:9113']
```
**Reiniciamos Prometheus utilizando el siguiente comando:** sudo systemctl restart prometheus

**Por último verificamos el éxito usando en la terminal...**
sudo systemctl status nginx
<img width="1198" height="337" alt="imagen" src="https://github.com/user-attachments/assets/59fbaa69-7811-4656-8b1a-928bd89f71a6" />

sudo systemctl status nginx_exporter.service
<img width="1573" height="274" alt="imagen" src="https://github.com/user-attachments/assets/8521730c-598e-4c4e-abb2-28991c6f37a7" />

**Ambos dos comandos deben aparecer en verde/running para afirmar el éxito**

**Por últimopara comprobar el éxito final en Prometheus ponemos en el navegador** http://localhost:9090 **status > targets > job nginx_metrics en verde o UP.** 
<img width="1338" height="155" alt="imagen" src="https://github.com/user-attachments/assets/268fcc3b-62ea-41c5-a9cc-c212521eedf5" />

**Alertas propuestas**

Con estas dos alertas garantizamos la disponibilidad del servicio, y se implementarán dentro de Prometheus. Estas métricas se encargan y vigilan de forma exclusiva tanto el estado del servidor web como la integridad del sistema de monitorización:

      - alert: NginxServerDown
        expr: nginx_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "¡Servidor NGINX caído!"
          description: "El exporter está levantado, pero reporta que NGINX no responde en {{ $labels.instance }}."

     
      - alert: NginxExporterDown
        expr: up{job="nginx_metrics"} == 0
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "¡NGINX Exporter inaccesible!"
          description: "Prometheus no puede raspar las métricas del exporter en {{ $labels.instance }}."


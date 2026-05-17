# BLACKBOX EXPORTER
### ¿Qué hace blackbox?
Es un exporter encargado de monitorear metricas de red como HTTP, HTTPS, DNS, TCP Y ICMP. Con el puedes analizar el tiempo que tarda un endpoint en responder o si está caido con diferentes alertas configurables.

### Arquitectura
Prometheus --> Blackbox Exporter --> Pagina Web

### Instalación
  1. mkdir blackbox
  2. cd blackbox
  3. wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.28.0/blackbox_exporter-0.28.0.linux-amd64.tar.gz
  4. tar -xvf blackbox_exporter-0.28.0.linux-amd64.tar.gz
  5. sudo nano /etc/systemd/system/blackbox_exporter.service
    
    [Unit]
    Description=Prometheus Blackbox Exporter
    Wants=network-online.target
    After=network-online.target
    
    [Service]
    User=ubuntu
    Type=simple
    ExecStart=/home/ubuntu/blackbox/blackbox_exporter-0.28.0.linux-amd64/blackbox_exporter \
      --config.file=/home/ubuntu/blackbox/blackbox_exporter-0.28.0.linux-amd64/blackbox.yml \
      --web.listen-address=":9115"

    [Install]
    WantedBy=multi-user.target
    
  <img width="1149" height="543" alt="image" src="https://github.com/user-attachments/assets/13f24e49-23ca-4251-b722-e368e285e9ba" />
  
  6. sudo systemctl daemon-reload
  7. sudo systemctl enable blackbox_exporter.service
  8. sudo systemctl start blackbox_exporter.service
  9. sudo systemctl status blackbox_exporter.service

### Métricas
#### probe_success
Tipo: Gauge\
Mide: Dependiendo de si la prueba de actividad del endpoint es exitosa devuelve un 1 (up) o 0 (down)\
Nuetra elección: Nos permitirá saber si nuestra web esta activa y así poder actuar rapidamente en el momento de su caida.

#### probe_duration_seconds
Tipo: Gauge\
Mide: Mostrará el tiempo de respuesta que tiene la pagina web en segundos\
Nuetra elección: Podremos ver el tiempo de respuesta que tiene la página para prevenir posibles caidas y detectar colisiones, cuellos de botella o obstrucciones en el intercambio de solicitudes.

#### probe_ssl_earliest_cert_expiry
Tipo: Gauge\
Mide: Calcula la fecha y hora exacta en el que caducara el certificado SSL/TLS\
Nuetra elección: Asegurará la identidad de la página web y el intercambio de información segura entre página y clientes. Sin el certificado todo eso no se cumpliría todo eso y podrian verse afectados datos, es por eso que nos avisará de su caducidad temprana.

### Inconveniente
No nos permitirá saber donde se encuentra los errores o problemas en el intercambio de paquetes y solamente nos muestra el resultado final

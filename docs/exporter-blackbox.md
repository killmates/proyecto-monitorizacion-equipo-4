BLACKBOX EXPORTER
1. Instalación
  1. mkdir blackbox
  2. cd blackbox
  3. wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.28.0/blackbox_exporter-0.28.0.linux-amd64.tar.gz
  4. tar -xvf blackbox_exporter-0.28.0.linux-amd64.tar.gz
  5. sudo nano /etc/systemd/system/blackbox_exporter.service
    Description=Prometheus Blackbox Exporter
    Wants=network-online.target
    After=network-online.target

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
  6. sudo systemctl daemon-reload
  7. sudo systemctl enable blackbox_exporter.service
  8. sudo systemctl start blackbox_exporter.service
  9. sudo systemctl status blackbox_exporter.service

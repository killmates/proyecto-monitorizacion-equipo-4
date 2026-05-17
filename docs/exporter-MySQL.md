# MYSQL EXPORTER

## ¿Qué es?

Es una herramienta que se conecta a MySQL y saca métricas para que Prometheus las pueda ver.

## Instalación

1. Instalar el servidor MySQL
   
   **sudo apt install mysql-server -y**

2. Encender el servidor de mysql
   
   **sudo systemctl enable --now mysql**

3. Entrar en sql y poner una serie de comandos para configurar usuarios y demas

   **-sudo mysql**
   
   **-CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'ContraseñaSegura!' WITH MAX_USER_CONNECTIONS 3;**
   
   **-GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';**
   
   **-FLUSH PRIVILEGES;**
   
   **-EXIT;**

4. Descargar el exporter

     
   **-sudo useradd --no-create-home --shell /bin/false mysqld_exporter**

   **-cd /tmp**
  
   **-wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.15.1/mysqld_exporter-0.15.1.linux-amd64.tar.gz**
  
   **-tar xvf mysqld_exporter-0.15.1.linux-amd64.tar.gz**
  
   **-sudo cp mysqld_exporter-0.15.1.linux-amd64/mysqld_exporter /usr/local/bin/**
  
   **-sudo chown mysqld_exporter:mysqld_exporter /usr/local/bin/mysqld_exporter**

5. Fichero con la contraseña

  **-sudo nano /etc/.mysqld_exporter.cnf** 

   Añadimos esto en el fichero 
   
      [client]
      
      user=exporter
      
      password=ContraseñaSegura!
      
      host=localhost

   Ajustamos permisos 
   
      sudo chown root:mysqld_exporter /etc/.mysqld_exporter.cnf
      
      sudo chmod 640 /etc/.mysqld_exporter.cnf

6. Crear el servicio para que arranque solo

  **-sudo nano /etc/systemd/system/mysqld_exporter.service**

    Añadimos esto en el fichero
    

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

  Arrancamos el servicio 

      sudo systemctl daemon-reload
      sudo systemctl enable --now mysqld_exporter

## COMPROBACION

 
  

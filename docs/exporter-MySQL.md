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
   

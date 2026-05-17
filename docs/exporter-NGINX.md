# ==> NGINX Exporter <==

**¿Que hace este Exporter?**
Extrae las métricas de rendimiento de un servidor NGINX y las traduce al formato compatible con Prometheus para que puedas monitorear tu servidor en tiempo real.

## Pasos a seguir para la istalación y configuración
**Ejecutamos en la terminal el siguiente comando para empezar con la instalación:** sudo apt update | sudo apt install nginx -y | sudo systemctl status nginx

**Ahora editamos el siguiente fichero:** sudo nano /etc/nginx/sites-available/default y añadimos las siguientes lineas...
location /nginx_status {
   stub_status on;
    allow 127.0.0.1;    * Solo permite que el exporter lea estos datos
    deny all;           * Prohíbe el acceso al resto del mundo
}

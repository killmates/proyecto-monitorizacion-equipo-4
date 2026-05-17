# Proyecto Monitorización Equipo 4

## Descripción

En este proyecto se monitoriza un servidor **Ubuntu Server 22.04** utilizando **Prometheus** y distintos exporters configurados por los miembros del equipo.

### Miembros del equipo

- **John** — Encargado del proyecto + HAProxy Exporter  
- **Soufian** — Node Exporter  
- **Oscar** — NGINX Exporter  
- **Leo** — Blackbox Exporter  
- **Rubén** — MySQL Exporter  

## Objetivo

Mediante el uso de estos exporters se supervisan métricas del sistema, servicios web, en este caso usaremos la app Wordpress que esta sostenin, disponibilidad de red, balanceo de carga y bases de datos en tiempo real.

Los exporters utilizados en el proyecto son:

- Node Exporter
- NGINX Exporter
- Blackbox Exporter
- HAProxy Exporter
- MySQL Exporter
## 3. Arquitectura

### Aplicación monitorizada

La aplicación desplegada en el servidor Ubuntu Server está formada por los siguientes componentes:

- HAProxy como balanceador de carga
- NGINX como servidor web
- PHP-FPM para la ejecución de PHP
- WordPress como aplicación web
- MySQL como base de datos

```text
Usuarios
   |
   v
HAProxy :8080
   |
   v
NGINX :80
   |
   v
PHP-FPM 8.1
   |
   v
WordPress
   |
   v
MySQL :3306
```

---

### Stack de observabilidad

Prometheus recoge métricas de diferentes exporters configurados en el servidor.

```text
                Prometheus :9090
                       |
     -----------------------------------
     |              |                 |
     v              v                 v
node_exporter   blackbox_exp   mysqld_exporter
    :9100           :9115            :9104
       |                |
       v                v
nginx_exporter    haproxy_exporter
     :9117              :9101
```

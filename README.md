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
## 4. Cómo se levanta el sistema

### 1. Actualizar el sistema

```bash
sudo apt update && sudo apt upgrade -y
```

---

### 2. Crear estructura de monitorización

```bash
sudo mkdir -p /opt/monitoring
```

---

### 3. Instalar Prometheus

Crear usuario:

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

Crear estructura:

```bash
sudo mkdir -p /opt/monitoring/prometheus/{bin,config,data}
```

Descargar Prometheus:

```bash
cd /tmp

wget https://github.com/prometheus/prometheus/releases/download/v3.11.3/prometheus-3.11.3.linux-amd64.tar.gz
```

Descomprimir:

```bash
tar xvf prometheus-3.11.3.linux-amd64.tar.gz
```

Copiar binarios:

```bash
sudo cp prometheus promtool /opt/monitoring/prometheus/bin/
```

---

### 4. Configurar Prometheus

Editar configuración:

```bash
sudo nano /opt/monitoring/prometheus/config/prometheus.yml
```

Añadir los targets de:

- node_exporter
- blackbox_exporter
- mysqld_exporter
- nginx_exporter
- haproxy_exporter

---

### 5. Crear servicio Prometheus

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Iniciar servicio:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```

---

### 6. Instalar exporters

#### Node Exporter

```bash
sudo useradd -rs /bin/false node_exporter
```

#### Blackbox Exporter

```bash
sudo useradd -rs /bin/false blackbox_exporter
```

#### MySQL Exporter
Configurado para monitorizar MySQL y WordPress.

#### NGINX Exporter
Configurado para monitorizar el servidor web NGINX.

#### HAProxy Exporter
Configurado para monitorizar estadísticas del balanceador.

---

### 7. Instalar WordPress

Instalar:
- MySQL
- PHP 8.1
- WordPress
- NGINX

Configurar WordPress y conectar con MySQL.

---

### 8. Instalar HAProxy

```bash
sudo apt install -y haproxy
```

Configurar frontend y backend en:

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

---

### 9. Instalar Grafana

```bash
sudo apt install -y grafana
```

Iniciar:

```bash
sudo systemctl enable --now grafana-server
```

---

### 10. Verificar funcionamiento

#### WordPress
```text
http://IP_SERVIDOR
```

#### HAProxy
```text
http://IP_SERVIDOR:8080
```

#### Prometheus
```text
http://IP_SERVIDOR:9090
```

#### Grafana
```text
http://IP_SERVIDOR:3000
```
## 5. Decisiones tomadas

Hemos elegido Node Exporter para monitorizar recursos del sistema como CPU, memoria RAM, disco y red, ya que es uno de los exporters más utilizados y ofrece métricas muy completas del servidor.

Utilizamos NGINX para supervisar el estado y rendimiento del servidor web NGINX, permitiendo controlar conexiones, peticiones y actividad del servicio web.

Tambien configuramos Blackbox porque permite comprobar disponibilidad y tiempos de respuesta con peticiones HTTP, simulando verificaciones reales desde el exterior.

Escogimos HAProxy para monitorizar el balanceador de carga y comprobar el estado de los backends y conexiones activas.

Por último utilizamos MySQL Exporter para supervisar el rendimiento de la base de datos y detectar posibles problemas de consultas o conexiones.

Los puertos utilizados corresponden principalmente a los valores por defecto de cada servicio y exporter, para así facilitar la configuración y compatibilidad con Prometheus:

- Prometheus → 9090
- Node Exporter → 9100
- HAProxy Exporter → 9101
- MySQL Exporter → 9104
- Blackbox Exporter → 9115
- NGINX Exporter → 9117
- Grafana → 3000
- HAProxy → 8080
- MySQL → 3306

No elegimos otros exporters porque no eran necesarios para los objetivos que teniamos y los escogidos cuadraban mñas con nuestros intereses

## 6. Limitaciones conocidas


- Algunos exporters requieren configuración manual adicional dependiendo del entorno.
- No se han implementado alertas avanzadas ni notificaciones automáticas mediante correo o Discord.
- Los dashboards de Grafana son básicos y dan bastantes fallos
- Si alguno de los exporters deja de responder, Prometheus dejará de recibir métricas de ese servicio.
- El sistema monitoriza principalmente infraestructura y servicios, pero no el rendimiento interno completo de aplicaciones como WordPress.
  
## 7. Fuentes consultadas
- Repositorios y apuntes de Alfonso Mariano de Uña del Brio para instalacion de prometheus y entender el funcionamiento de grafana
- Prometheus Official documentacion 
- Videos de youtube para fallos e instalacion y dashboards de grafana
- Ayuda adicional generada con ChatGPT/OpenAI, Gemini, Claude para configuración, resolución de errores y creación de dashboards y alertas Prometheus/Grafana.

# HAProxy Exporter

## 1. Identificación

Nombre y apellidos: John Huilca

Exporter trabajado: HAProxy Exporter

---

## 2. Qué hace este exporter

HAProxy Exporter permite monitorizar el estado y rendimiento del balanceador de carga HAProxy desde Prometheus. Gracias a este exporter se pueden supervisar conexiones activas, tráfico HTTP, estado de los backends y número de peticiones recibidas.

Este exporter ayuda a detectar caídas de servicios, saturación del balanceador o problemas de conectividad entre el frontend y los servidores backend.

---

## 3. Cómo se instala

### Instalación de HAProxy Exporter

Crear usuario:

```bash
sudo useradd --no-create-home --shell /bin/false haproxy_exporter
```

Crear estructura:

```bash
sudo mkdir -p /opt/monitoring/haproxy_exporter/bin
```

Descargar exporter:

```bash
cd /tmp

wget https://github.com/prometheus/haproxy_exporter/releases/download/v0.15.0/haproxy_exporter-0.15.0.linux-amd64.tar.gz
```

Descomprimir:

```bash
tar -xvf haproxy_exporter-0.15.0.linux-amd64.tar.gz
```

Entrar en la carpeta:

```bash
cd haproxy_exporter-0.15.0.linux-amd64
```

Copiar binario:

```bash
sudo cp haproxy-exporter /opt/monitoring/haproxy_exporter/bin/
```

Asignar permisos:

```bash
sudo chown -R haproxy_exporter:haproxy_exporter /opt/monitoring/haproxy_exporter
```

---

### Configuración de HAProxy

Editar configuración:

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Añadir:

```cfg
global
    daemon
    stats socket /run/haproxy/admin.sock mode 660 level admin

defaults
    mode http

    timeout connect 5s
    timeout client 50s
    timeout server 50s

frontend http_front
    bind *:8080

    default_backend nginx_back

backend nginx_back

    balance roundrobin

    server nginx1 127.0.0.1:80 check

listen stats

    bind *:8404

    stats enable
    stats uri /stats
```

Reiniciar HAProxy:

```bash
sudo systemctl restart haproxy
```
<img width="1757" height="373" alt="imagen" src="https://github.com/user-attachments/assets/3b11acfe-3489-4c9b-8ccc-0819bd42d633" />

---

### Crear servicio systemd del exporter

Crear servicio:

```bash
sudo nano /etc/systemd/system/haproxy_exporter.service
```

Contenido:

```ini
[Unit]
Description=HAProxy Exporter
After=network.target

[Service]
User=haproxy_exporter
Group=haproxy_exporter

ExecStart=/opt/monitoring/haproxy_exporter/bin/haproxy-exporter \
  --haproxy.scrape-uri="unix:/run/haproxy/admin.sock"

Restart=always

[Install]
WantedBy=multi-user.target
```

Activar exporter:

```bash
sudo systemctl daemon-reload

sudo systemctl enable --now haproxy_exporter

sudo systemctl status haproxy_exporter
```

---

### Configuración en Prometheus

Editar:

```bash
sudo nano /opt/monitoring/prometheus/config/prometheus.yml
```

Añadir:

```yaml
- job_name: "haproxy"
  static_configs:
    - targets: ["localhost:9101"]
```

Reiniciar Prometheus:

```bash
sudo systemctl restart prometheus
```

---
<img width="1762" height="264" alt="imagen" src="https://github.com/user-attachments/assets/2fb8f5e2-c4a3-4d46-877c-8df9fbeccc5a" />


### Verificar métricas

Abrir en navegador:

```text
<img width="1859" height="977" alt="imagen" src="https://github.com/user-attachments/assets/65d9af4d-ba32-4f47-8dda-b352fdf6edcf" />

```
## 4. Tres métricas clave

### 1. haproxy_frontend_current_sessions

- **Tipo:** Gauge  
- **Qué mide:** Número actual de sesiones activas en el frontend de HAProxy.  
- **Por qué se ha elegido:** Permite conocer la carga real que está soportando el balanceador en tiempo real y detectar posibles saturaciones

---

### 2. haproxy_frontend_http_requests_total

- **Tipo:** Counter  
- **Qué mide:** Número total de peticiones HTTP procesadas por HAProxy.  
- **Por qué se ha elegido:** Permite analizar el tráfico, calcular peticiones por segundo y detectar aumentos anormales de actividad

---

### 3. haproxy_up

- **Tipo:** Gauge  
- **Qué mide:** Indica si HAProxy está operativo.  
  - `1` → funcionando  
  - `0` → caído  
- **Por qué se ha elegido:** Es una métrica esencial para comprobar rápidamente si el servicio está activo o ha dejado de funcionar
## 5. Una alerta propuesta

```yaml
- alert: HAProxyDown
  expr: haproxy_up == 0
  for: 1m

  labels:
    severity: critical

  annotations:
    summary: "HAProxy DOWN"
    description: "HAProxy exporter no responde"
```

Esta alerta detecta cuando el servicio HAProxy deja de responder durante más de 1 minuto. Se ha configurado con severidad crítica porque la caída del balanceador puede impedir el acceso a los servicios web monitorizados.

  

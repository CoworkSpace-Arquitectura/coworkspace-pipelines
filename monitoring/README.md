# Monitoreo de CoworkSpace con Prometheus y Grafana

Cada microservicio usa la librería **prom-client** y publica sus métricas en `GET /metrics`.
**Prometheus** las consulta cada 15 segundos y **Grafana** las muestra en un dashboard.
Prometheus y Grafana **no se despliegan**: se ejecutan en un PC para la demostración y las capturas.

## Qué se mide

| Señal | Métrica | Dónde se ve |
|---|---|---|
| Disponibilidad | `up` (1 = responde, 0 = caído) | Paneles *Servicios en línea* y *Estado de cada microservicio* |
| Número de peticiones | `http_requests_total{method, route, status_code}` | *Peticiones por segundo*, *Rutas más usadas* |
| Tiempo de respuesta | `http_request_duration_seconds` (histograma) | *Tiempo de respuesta (percentil 95)* |
| Tasa de errores | `http_requests_total{status_code=~"5.."}` | *Tasa de errores 5xx*, *Respuestas por código HTTP* |
| Saturación | `process_resident_memory_bytes`, `process_cpu_seconds_total` | *Memoria* y *CPU* |
| Negocio | `coworkspace_bookings_created_total`, `coworkspace_payments_total` | *Reservas creadas*, *Pagos con Stripe* |

Alertas (`alerts.yml`): **ServicioCaido** (`up == 0` por 2 min), **MuchosErrores** (> 5 % de 5xx) y **RespuestaLenta** (p95 > 2 s).

## Archivos

| Archivo | Para qué |
|---|---|
| `prometheus.yml` | Targets: los 10 microservicios desplegados en Render (`https://…/metrics`) |
| `prometheus.local.yml` | Variante para cuando los servicios corren en el PC con `npm run start:dev` |
| `alerts.yml` | Reglas de alerta |
| `docker-compose.yml` | Levanta Prometheus y Grafana con un comando |
| `grafana/provisioning/` | Conecta Grafana con Prometheus y carga el dashboard solos |
| `grafana/dashboards/coworkspace.json` | El dashboard (se puede importar a mano) |

## Opción A: con Docker

```bash
cd monitoring
docker compose up -d
```

- Prometheus: http://localhost:9090 → **Status → Targets**: los 10 servicios deben estar en **UP**.
- Grafana: http://localhost:3000 (admin / admin) → **Dashboards → CoworkSpace → Estado de los microservicios**.

## Opción B: en Windows sin Docker

1. **Prometheus**: descarga `prometheus-3.5.0.windows-amd64.zip` de https://prometheus.io/download/ y descomprímelo.
   Copia `prometheus.yml` y `alerts.yml` de esta carpeta dentro de la carpeta descomprimida (reemplaza el que trae) y ejecuta en CMD:
   ```
   prometheus.exe --config.file=prometheus.yml
   ```
   Abre http://localhost:9090/targets.
2. **Grafana**: instala Grafana OSS para Windows desde https://grafana.com/grafana/download?platform=windows y abre http://localhost:3000 (admin / admin).
   - **Connections → Data sources → Add data source → Prometheus**, URL `http://localhost:9090`, **Save & test**.
   - **Dashboards → New → Import → Upload JSON file** y elige `grafana/dashboards/coworkspace.json`.

## Antes de la demostración

Los servicios de Render se duermen tras 15 minutos sin uso. Ejecuta el workflow **despertar-servicios**
(Actions → despertar-servicios → Run workflow) y espera a que los targets pasen a **UP**.
Para que las gráficas tengan datos, usa el sitio unos minutos (buscar sedes, reservar, pagar).

Al terminar, apaga Prometheus: si sigue consultando, mantiene despiertos los servicios y gasta las horas gratis de Render.

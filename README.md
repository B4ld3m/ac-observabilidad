# Laboratorio de Observabilidad

## Autor

**Rodrigo Alexander Baldeón Julca**

---

## Comandos utilizados

```bash
docker compose up -d --build     # levantar / reconstruir
docker compose ps                # estado de los servicios
docker compose logs -f grafana   # seguir logs de un servicio
docker compose down              # detener (conserva dashboards)
docker compose down -v           # detener y borrar todos los datos
```

---

## Actividades realizadas

### 1. Levantamiento del stack

Se ejecutó el siguiente comando para construir las imágenes y levantar todos los servicios:

```bash
docker compose up -d --build
```

Luego se verificó el estado de los contenedores:

```bash
docker compose ps
```

Se comprobó el acceso a:

- Frontend: http://localhost:8080
- Backend: http://localhost:3001/metrics
- Grafana: http://localhost:3000
- Prometheus: http://localhost:9090

---

### 2. Generación de tráfico y logs

Desde el frontend se realizaron múltiples solicitudes mediante el botón **"Saludar (API)"** para generar métricas y registros de actividad.

También se utilizó el botón **"Generar carga de CPU (30s)"** para provocar consumo de CPU y probar las alertas configuradas.

---

### 3. Verificación de fuentes de datos

Se ingresó a Grafana y se verificó que las fuentes de datos aprovisionadas estuvieran disponibles:

- Prometheus
- Loki

Ambas fueron detectadas correctamente sin necesidad de configuración manual.

---

### 4. Construcción del dashboard

Se creó un dashboard de observabilidad con los siguientes paneles:

#### CPU del contenedor backend

```promql
sum(rate(container_cpu_usage_seconds_total{name="lab-backend"}[1m])) * 100
```

#### CPU del host

```promql
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
```

#### Logs de aplicación

```logql
{tier="application"} | json
```

#### Logs de infraestructura

```logql
{tier="infrastructure"}
```

Finalmente se guardó el dashboard con el nombre:

```text
Observabilidad — Rodrigo Alexander Baldeón Julca
```

---

### 5. Configuración de alerta de CPU

Se creó una regla de alerta llamada:

```text
CPU backend > 50%
```

Utilizando la consulta:

```promql
sum(rate(container_cpu_usage_seconds_total{name="lab-backend"}[1m])) * 100
```

Configuración:

- Threshold: > 50
- Evaluation interval: 10s
- Pending period: 30s
- Severity: warning

---

### 6. Prueba de la alerta

Se generó carga de CPU desde el frontend para superar el umbral configurado.

Durante la prueba se verificó la transición de estados:

```text
Normal → Pending → Firing
```

Además, en los logs de infraestructura se observaron eventos relacionados con el sistema de alertas, tales como:

```text
Sending alerts to local notifier
```

confirmando el procesamiento de las notificaciones por parte de Grafana.

---

## Preguntas

### ¿Por qué necesitamos Loki además de Prometheus si ya tenemos /metrics?

Prometheus almacena métricas numéricas, mientras que Loki almacena logs. Los logs permiten ver errores y eventos específicos que las métricas no muestran.

### ¿Qué ventaja aporta que las fuentes de datos de Grafana estén aprovisionadas como código y no creadas a mano?

Permite tener configuraciones reproducibles, reducir errores humanos y automatizar los despliegues.

### El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué?

Porque el CPU del contenedor muestra el uso de una aplicación específica, mientras que el CPU del host muestra el consumo total de la máquina.

### ¿Cuál usarías para alertar sobre una aplicación concreta?

El CPU del contenedor, porque mide directamente el consumo de la aplicación monitoreada.

### ¿Qué diferencia hay entre el evaluation interval y el pending period de una alarma?

- Evaluation interval: cada cuánto tiempo se evalúa la alerta.
- Pending period: cuánto tiempo debe mantenerse la condición antes de disparar la alerta.



## Servicios y URLs
| Servicio       | URL                         | Notas                                  |
|----------------|-----------------------------|----------------------------------------|
| Frontend       | http://localhost:8080       | Hello World + botones de tráfico/carga |
| Backend (API)  | http://localhost:3001       | `/api/hello`, `/metrics`, `/load`      |
| Grafana        | http://localhost:3000       | admin / admin                          |
| Prometheus     | http://localhost:9090       | datasource ya provisionado             |
| Loki           | http://localhost:3100       | datasource ya provisionado             |
| Alloy (UI)     | http://localhost:12345      | estado del recolector de logs          |
| cAdvisor       | http://localhost:8081       | métricas por contenedor                |
| node-exporter  | http://localhost:9100/metrics | métricas del host                    |

## Configuraciones
- **Datasources** Prometheus y Loki (provisionados automáticamente).
- Logs etiquetados por Alloy con `tier=application` o `tier=infrastructure`.

## Actividad
- El **dashboard** (paneles de CPU + logs de app e infra).
- La **alarma** de CPU > 50%.

## Reset
```bash
docker compose down -v   # borra también dashboards/alarmas creados
```

> Nota de versiones: el tag `prom/prometheus:latest` apunta aún a la rama 2.x (LTS),
> por eso fijamos `v3.8.1`. Promtail EOL (2026-03-02); el recolector de logs
> es Grafana Alloy.


## Preguntas y respuestas

### ¿Por qué necesitamos Loki además de Prometheus si ya tenemos /metrics?

Prometheus almacena métricas numéricas, mientras que Loki almacena logs. Los logs permiten ver errores y eventos específicos que las métricas no muestran.

### ¿Qué ventaja aporta que las fuentes de datos de Grafana estén aprovisionadas como código y no creadas a mano?

Permite tener configuraciones reproducibles, reducir errores humanos y automatizar los despliegues.

### El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué?

Porque el CPU del contenedor muestra el uso de una aplicación específica, mientras que el CPU del host muestra el consumo total de la máquina.

### ¿Cuál usarías para alertar sobre una aplicación concreta?

El CPU del contenedor, porque mide directamente el consumo de la aplicación monitoreada.

### ¿Qué diferencia hay entre el evaluation interval y el pending period de una alarma?

- Evaluation interval: cada cuánto tiempo se evalúa la alerta.
- Pending period: cuánto tiempo debe mantenerse la condición antes de disparar la alerta.
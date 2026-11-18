# coworkspace-pipelines

Infraestructura compartida de CoworkSpace.

- `.github/workflows/nest-service.yml`: pipeline **reutilizable** (`workflow_call`) que usan los 10 microservicios:
  1. **Pruebas**: instala dependencias, crea una base PostgreSQL y un Redis de prueba, aplica las migraciones,
     ejecuta lint, build, pruebas unitarias y de integración, y genera el **reporte de cobertura** (artefacto `cobertura-<servicio>`).
  2. **Imagen Docker**: solo en `main` y solo si las pruebas pasan; la publica en GHCR.
  3. **Despliegue**: llama al deploy hook de Render (secreto `RENDER_DEPLOY_HOOK_URL`).
- `.github/workflows/despertar-servicios.yml`: despierta los servicios de Render antes de una demostración.
- `monitoring/`: Prometheus y Grafana (ver [monitoring/README.md](monitoring/README.md)).

## Estrategia de ramas

- `main` es la rama de producción: cada push a `main` ejecuta el pipeline completo y, si las pruebas pasan, despliega.
- Los cambios se trabajan en ramas `feature/<nombre>` y se integran con un pull request: el PR ejecuta las pruebas
  (sin desplegar) y solo se fusiona si están en verde.
- Si una prueba falla, el job de pruebas falla y los jobs de imagen y despliegue no se ejecutan.

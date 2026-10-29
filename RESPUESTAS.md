# Respuestas · Laboratorio 4
Alumno: 20216686
Presupuesto: 13 min  Métrica DORA:  time to restore

## Parte 1 · Diagnóstico
### Defecto 1
- Job y etapa: smoke, etapa deploy (debe estar en post-deploy).
- Problema que causa en el pipeline: El job smoke corría en paralelo con deploy (needs: build-push), haciendo que las pruebas de humo consultaran la Lambda antes de que el despliegue del nuevo código hubiera finalizado.
- Link al commit: https://github.com/JuanBa15/INF384-lab4-dev-20216686/commit/eff4202c9b2d28727249261898d1a6bab0cda3c2
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): mover de etapa
- Evidencia (dato observado, con link al run o fragmento de la salida local): 
### Defecto 2
- Job y etapa: integration, etapa pre-build.
- Problema que causa en el pipeline: Falta de aislamiento en /tmp/wallet.db. Las pruebas modifican saldos sin reiniciar la BD, provocando fallos en ejecuciones consecutivas por acumulación de estado.
- Link al commit: https://github.com/JuanBa15/INF384-lab4-dev-20216686/commit/d61e2f1cb67c4b642ec52ae7e0dc768c8cb41f32
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Modificar tests
- Evidencia (dato observado, con link al run o fragmento de la salida local):
### Defecto 3
- Job y etapa: e2e, etapa post-deploy.
- Problema que causa en el pipeline: Descarga e instalación redundante de Chromium (`npx playwright install --with-deps chromium`), desperdiciando ~1.5 min en un suite que evalúa peticiones HTTP REST.
- Link al commit: https://github.com/JuanBa15/INF384-lab4-dev-20216686/commit/3f3a5d7ff84977ad7a22c710545d2bde5b1c7742    https://github.com/JuanBa15/INF384-lab4-dev-20216686/commit/4189223bce9a1252d286df99662306e461cfbc83
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Eliminar / Modificar tests
- Evidencia (dato observado, con link al run o fragmento de la salida local):
### Defecto 4
- Job y etapa: load, etapa post-deploy.
- Problema que causa en el pipeline: La etapa de carga constante en k6 estaba configurada en 10 minutos (10m), desperdiciando 6 minutos del presupuesto de tiempo de ejecución del pipeline.
- Link al commit: https://github.com/JuanBa15/INF384-lab4-dev-20216686/commit/c1611b50d8710ddcdb617c355f451f8bab0933fc
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Modificar tests
- Evidencia (dato observado, con link al run o fragmento de la salida local):
### Defecto 5
- Job y etapa: smoke, e2e, load, etapa post-deploy.
- Problema que causa en el pipeline: Tiempos de retardo excesivos (sleep 10 y sleep 5) durante la resolución de URL de Lambda y peticiones de salud, demorando innecesariamente la fase post-despliegue.
- Link al commit: https://github.com/JuanBa15/INF384-lab4-dev-20216686/commit/53e1246f6b75dff22f8d02ead1ef9fa676a40a25      https://github.com/JuanBa15/INF384-lab4-dev-20216686/commit/979065ca5c51b008f3c9b000fdc68936cf1a24ac
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Ajustar configuración / Modificar tests
- Evidencia (dato observado, con link al run o fragmento de la salida local):

## Parte 2 · Implementación
- Link al run en verde en main:
- Link al pull request:
- Defecto de la aplicación introducido (archivo y descripción):
- Job que debería detenerlo:
- Link al run fallido:

## Parte 3 · Decisión bajo restricción
Link al run medido:

| Job | Duración (min:seg) |
|---|---|
| unit | |
| integration | |
| sast | |
| sca | |
| build-push | |
| deploy | |
| smoke | |
| e2e | |
| load | |
| **Total** | |

Resultado de load en ese run:
| Métrica | Valor |
|---|---|
| p90 | |
| p95 | |
| Máximo | |
| Tasa de error | |
| Requests por segundo | |

Conclusión sobre el comportamiento bajo carga:

SLO propuesto (percentil, valor, ventana) y justificación:

SLA propuesto y diferencia con el SLO:

Cambios para cumplir el presupuesto:
| Cambio | Job | Minutos recuperados |
|---|---|---|

Total resultante: [ ] min
Link al run en main con los cambios:

Riesgo aceptado (defecto de la aplicación concreto y efecto en la métrica DORA asignada):

Decisión sobre sast (se queda / se mueve a otra etapa o evento / informativo) y razón:

## Declaración de uso de IA
Herramientas usadas y para qué:

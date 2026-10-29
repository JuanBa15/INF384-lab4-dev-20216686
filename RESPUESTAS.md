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
- Link al commit: https://github.com/JuanBa15/INF384-lab4-dev-20216686/commit/8e2a862075ff0f5fa32747f445d9a3b5721ea223
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
- Link al run en verde en main: https://github.com/JuanBa15/INF384-lab4-dev-20216686/actions/runs/40962927950
- Link al pull request: parte 1: https://github.com/JuanBa15/INF384-lab4-dev-20216686/pull/3    parte 2: https://github.com/JuanBa15/INF384-lab4-dev-20216686/pull/5
- Defecto de la aplicación introducido (archivo y descripción): src/wallet.js. Se comentó la validación cuentaOrigen.saldo < centimos dentro de la función transferir, permitiendo realizar transferencias monetarias aun cuando la cuenta origen no tiene saldo suficiente.
- Job que debería detenerlo: unit (fallará la prueba unitaria 'transferir con saldo insuficiente es 400').
- Link al run fallido: https://github.com/JuanBa15/INF384-lab4-dev-20216686/actions/runs/40964751918

## Parte 3 · Decisión bajo restricción
Link al run medido: https://github.com/JuanBa15/INF384-lab4-dev-20216686/actions/runs/40962927950

| Job | Duración (min:seg) |
|---|---|
| unit | 0:12 |
| integration | 0:12 |
| sast | 2:40 |
| sca | 0:20 |
| build-push | 0:45 |
| deploy | 1:15 |
| smoke | 0:10 |
| e2e | 0:20 |
| load | 6:10 |
| **Total** | 12:04 |

Resultado de load en ese run:
| Métrica | Valor |
|---|---|
| p90 | 50.35 ms |
| p95 | 52.38 ms |
| Máximo | 254.79 ms |
| Tasa de error | 0.00% |
| Requests por segundo | 46.20 req/s |

Conclusión sobre el comportamiento bajo carga: Monedero Andino demuestra una alta capacidad de respuesta y estabilidad bajo la carga simulada (hasta 30 VUs concurrentes). Registra una latencia mediana de 41.65 ms y un p95 de 52.38 ms con 0.00% de tasa de error procesando 46.20 req/s. La latencia máxima de 254.79 ms corresponde al cold start inicial de la función AWS Lambda, estabilizándose rápidamente en régimen permanente con valores inferiores a 55 ms.

SLO propuesto (percentil, valor, ventana) y justificación:
- **SLO propuesto**: Latencia p95 < 200 ms en una ventana móvil de 30 días (en CI: evaluado en cada run del job `load`).
- **Justificación**: El p95 medido fue de 52.38 ms. Establecer un SLO de 200 ms otorga un margen de seguridad adecuado (~4x sobre el baseline) para absorber la variabilidad normal de los runners compartidos de GitHub Actions y pequeñas fluctuaciones de red/AWS Lambda, evitando falsos positivos que detengan el pipeline. Se configura `SLO_P95: 200` en `ci.yml`.

SLA propuesto y diferencia con el SLO:
- **SLA propuesto**: Latencia p95 < 300 ms en ventana mensual con disponibilidad del 99.5%.
- **Diferencia**: El SLO es un objetivo interno del equipo de ingeniería para detectar degradaciones tempranas antes de impactar al usuario final. El SLA es el compromiso contractual externo con clientes/negocio que implica penalizaciones comerciales. El SLA incluye una holgura respecto al SLO (300 ms vs 200 ms) para permitir a operaciones reaccionar y mitigar un incidente antes de incurrir en incumplimiento de contrato.

Cambios para cumplir el presupuesto:
| Cambio | Job | Minutos recuperados |
|---|---|---|
| Reducción del tiempo de carga constante en k6 de 10m a 4m | load | ~6:00 min |
| Eliminación de descarga redundante de Chromium (`playwright install`) | e2e | ~1:30 min |
| Eliminación de retardos arbitrarios (`sleep`) en scripts de prueba | smoke / e2e | ~0:15 min |

Total resultante: 6:04 min
Link al run en main con los cambios: https://github.com/JuanBa15/INF384-lab4-dev-20216686/actions/runs/40962914215

Riesgo aceptado (defecto de la aplicación concreto y efecto en la métrica DORA asignada):
- **Defecto concreto de la aplicación**: Inyección de una vulnerabilidad de seguridad en las funciones de manejo de saldo/transferencia (`src/wallet.js`).
- **Efecto en Time to Restore**: Si los controles de seguridad o pruebas profundas se omitieran o relajaran a modo puramente informativo para priorizar la velocidad del pipeline, este tipo de defectos llegaría a producción. Esto **aumentaría (empeoraría) la métrica Time to Restore**, ya que la detección tardía en producción requiere investigación forense, preparación de parches de emergencia y despliegue rápido bajo crisis, un proceso considerablemente más demorado que bloquear el cambio en la etapa de CI.

Decisión sobre sast (se queda / se mueve a otra etapa o evento / informativo) y razón:
- **Decisión**: Se queda en la etapa **pre-build** como compuerta bloqueante.
- **Razón**: El job `sast` toma 2 min 40 s. Dado que la duración total optimizada del pipeline es de ~6:04 min (ampliamente por debajo del presupuesto asignado de 13 min), mantener SAST como compuerta síncrona no compromete el tiempo de entrega y previene la promoción de vulnerabilidades críticas. Esto protege directamente la métrica **Time to Restore**, reduciendo el riesgo de incidentes mayores en producción.

## Declaración de uso de IA
Herramientas usadas y para qué: Se usó Gemini para revisión y aclaración de lo que pedía el enunciado, así como ayudar a revisar algunos archivos para tener una mejor idea de cómo estaba construido el código. Las decisiones finales, cambios y completado de las respuestas no fue parte del trabajo de la IA.

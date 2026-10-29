# Respuestas · Laboratorio 4
Alumno: 20216686
Presupuesto: [ ] min  Métrica DORA: [ ]

## Parte 1 · Diagnóstico
### Defecto 1: Falta de ejecución en paralelo entre las pruebas iniciales
- Job y etapa: sast y sca, o puede que sea la dependencia entre ellos, en la etapa pre-build
- Problema que causa en el pipeline: Falta de optimización en el flujo de ejecución en paralelo de los análisis iniciales, lo que retrasa la entrada al job `build-push`.
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Ajustar configuración
- Evidencia (dato observado, con link al run o fragmento de la salida local): 
### Defecto 2: Las pruebas de integración se ejecutan sin aislamiento en la base de datos
- Job y etapa: integration en la etapa pre-build.
- Problema que causa en el pipeline: Acoplamiento de estado en la base de datos SQLite entre pruebas sin limpieza de datos, ocasionando resultados no deterministas.
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Modificar tests
- Evidencia (dato observado, con link al run o fragmento de la salida local):
### Defecto 3: Instalación redundante o innecesaria de navegadores en test:e2e
- Job y etapa: e2e en la etapa post-deploy.
- Problema que causa en el pipeline: Descarga e instalación innecesaria del navegador Chromium (`playwright install`) para pruebas que son 100% de API HTTP, desperdiciando tiempo del pipeline.
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Eliminar / Ajustar configuración
- Evidencia (dato observado, con link al run o fragmento de la salida local):
### Defecto 4: Script de pruebas de humo sin tolerancia a fallos transitorios / bucle de espera bloqueante
- Job y etapa: smoke en la etapa post-deploy.
- Problema que causa en el pipeline: Tiempos de espera excesivos en los reintentos de polling del script `smoke.sh` y en la obtención de la URL de Lambda, ralentizando la verificación post-despliegue.
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Ajustar configuración / Modificar tests
- Evidencia (dato observado, con link al run o fragmento de la salida local):
### Defecto 5: Redundancia de comandos y falta de uso eficiente del script en package.json
- Job y etapa: unit e integration en la etapa pre-build.
- Problema que causa en el pipeline: Descarga redundante e instalación desacoplada de dependencias en cada job individual sin optimizar el uso de caché o scripts del paquete.
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Ajustar configuración
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

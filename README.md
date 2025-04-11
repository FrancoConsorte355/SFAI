# Plan de Monitoreo y Análisis de Logging para LibreChat

## 1. Investigación de las capacidades de Logging en LibreChat

### Objetivo del Caso de Uso
Implementar un sistema de trazabilidad y análisis que:
- Categoriza automáticamente las preguntas realizadas por los usuarios internos.
- Relacione cada pregunta con el documento o fuente consultada.
- Permite medir qué tan útil es cada documento y detectar brechas de conocimiento o duplicaciones.

### Modelo Conceptual (a nivel datos)

| Entidad      | Descripción                                                                 |
|-------------|-----------------------------------------------------------------------------|
| Pregunta     | Consulta realizada por un colaborador (usuario)                            |
| Categoría    | Categoría de la pregunta: Fullstack, Database, Backend, etc.               |
| Rol del usuario | Rol desde el cual se formula la pregunta (puede coincidir o complementar) |
| Documento    | Fuente que el motor utiliza para responder (puede ser doc técnico, README) |
| Log de uso   | Registro detallado del flujo (usuario, hora, éxito, error, sugerencia usada, etc.) |

## Columnas de la base de datos

- ID de consulta
- Nombre de usuario (en caso de no estar registrado — Null)
- Rol de usuario (Full Stack, Frontend, Backend, QA, Data Analyst)
- Texto de la consulta — Pregunta realizada
- Documentos consultados
- Tiempo de Respuesta
- Logs — Clasificación:
  - Emergency
  - Alert
  - Critical
  - Error
  - Warning
  - Notice
  - Informational
  - Debug
- Resultado:
  - Éxito
  - Error
  - Sin respuesta
  - Respuesta parcial
- Feedback del usuario:
  - Like
  - Regenerate
  - Valoración manual
- Versión del modelo/algoritmo — Identifica qué motor o modelo respondió la consulta

## 2. Métricas para monitorear la salud del servicio

| Métrica                          | Propósito                                          | Cálculo                                                    | Umbral de Alerta / Acción Recomendada                  |
|----------------------------------|----------------------------------------------------|-------------------------------------------------------------|---------------------------------------------------------|
| Preguntas por categoría          | Detectar qué áreas tienen mayor demanda de conocimiento | Conteo de preguntas agrupadas por categoría                 | > 40% en una categoría → revisar documentación existente |
| Preguntas por documento consultado | Identificar documentos más consumidos o útiles    | Conteo por documento enlazado por respuesta                | Documento con > 50 consultas/semana = "popular"         |
| Preguntas sin documento asociado | Detectar preguntas no respondidas con contenido actual | Conteo de respuestas sin documentos o sin respuesta         | > 5% por semana → indicar necesidad de generar documentación |
| Tiempo promedio de respuesta     | Medir eficiencia del sistema de sugerencias        | Diferencia entre hora de pregunta y respuesta              | > 5 segundos = revisar eficiencia del motor              |
| Ratio de re-preguntas (follow-up) | Verificar si las respuestas son claras o incompletas | % de preguntas con preguntas sucesivas del mismo usuario    | > 25% → posible brecha de claridad o documentación insuficiente |
| Categoría sin cobertura          | Detectar categorías sin respuestas válidas         | Categorías con 0 respuestas exitosas en un periodo         | Revisión semanal y creación de documentos               |
| Usuarios más activos (preguntadores) | Detectar necesidades de onboarding o mentoring     | Agrupación por usuario                                     | > 10 preguntas por semana → posible acompañamiento personalizado |
| Éxito de sugerencia por documento | Evaluar calidad de contenido sugerido              | % de respuestas con feedback positivo / sin re-pregunta por documento | < 70% de éxito → revisar documento                     |

### ¿Cómo instrumentarlo técnicamente?
Cada vez que un colaborador realiza una pregunta, registrar:
- ✅ El texto de la pregunta
- 🧩 La categoría
- 👤 El rol del usuario
- 📄 Los documentos consultados para responder
- 🕐 Tiempos de respuesta
- 📶 Resultado (respondido, con error, requiere nueva doc)

## 3. Recolección de las métricas propuestas

**Herramientas / métodos sugeridos:**
- LibreChat como backend generador de actividad.
- MongoDB Atlas como base de datos de almacenamiento.
- Grafana para visualización en tiempo real.

## 4. Plan Básico de Monitoreo

**Frecuencia:**
- Tiempo real: Dashboards en Grafana muestran métricas y logs en vivo.
- Revisión diaria: El equipo de DevOps revisa manualmente dashboards y logs.

**Herramientas:**
- **Grafana**: Visualización en tiempo real.
- **Prometheus**: Recolección de métricas.
- **Alertas automáticas**: Notificaciones por Slack, mail o generación de tickets en Jira.

**Procedimiento ante anomalías:**
- **Detección**: Se activa una alerta cuando alguna métrica excede su umbral (ej: latencia > 5s).
- **Análisis**: Revisión de logs recientes para confirmar la causa raíz.
- **Escalamiento**: Notificación al equipo técnico para aplicar medidas correctivas.

## 5. Escenario de Prueba de Monitoreo

*Escenario*:
 Simulación de respuestas lentas del sistema de sugerencias durante una carga de consultas moderada.
 
*Contexto*:
 Se detecta que el sistema presenta un tiempo promedio de respuesta de 6.2 segundos, superando el umbral de 5 segundos establecido para medir la eficiencia del motor de sugerencias. Las pruebas se realizan bajo una carga constante de 40 mensajes por segundo (MPS).
 
*Resultado Esperado*:
  -Alerta automática:
 Se dispara una alerta cuando el promedio de respuesta supera los 5 segundos durante un período de más de 1 minuto.

  -Logs y dashboards:
 Muestran la diferencia entre la hora de ingreso de la pregunta y la generación de la respuesta, evidenciando la latencia con ejemplos concretos. Se visualiza un patrón consistente de respuestas demoradas.

-Medidas correctivas:
 Se ejecutan acciones de revisión del motor de sugerencias, incluyendo:
 
    -Análisis de cuellos de botella en el pipeline de procesamiento.
    
    -Reentrenamiento o ajuste de hiperparámetros del modelo de sugerencias.
    
    -Optimización del cache o reducción de complejidad del algoritmo si aplica.


## 6. Claridad y Precisión
El documento está redactado con lenguaje técnico accesible, estructurado para su implementación clara por cualquier miembro del equipo técnico.

## 7. Revisión Ortográfica
Este documento ha sido revisado y está libre de errores ortográficos y gramaticales.



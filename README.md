# Voice of Customer (VoC) – Analítica Conversacional para Gestión de Cobranza

Prueba técnica para el rol de **Científico(a) de AI**.

Este proyecto analiza conversaciones de WhatsApp entre clientes y asesores de una operación de cobranza, utilizando técnicas de:

- Procesamiento de Lenguaje Natural (NLP)
- Inteligencia Artificial Generativa
- Prompt Engineering
- Analítica conversacional
- Visualización en Power BI
- Diseño de arquitectura RAG

El objetivo principal es transformar conversaciones no estructuradas en variables analíticas que permitan identificar motivos de no pago, estrategias de negociación, acuerdos de pago, experiencia del cliente y oportunidades de mejora para el negocio.

---

## Objetivo

Analizar conversaciones de cobranza para responder las siguientes preguntas de negocio:

1. ¿Cuáles son los principales motivos de no pago?
2. ¿Qué ofertas realizan los asesores para lograr acuerdos de pago?
3. ¿Qué ofrecimientos o argumentos presentan mayor tasa de acuerdos?
4. ¿Qué ocurre en cada conversación?
5. ¿Qué caracteriza las conversaciones con menor satisfacción o experiencia?

Adicionalmente, se propone una arquitectura basada en **Retrieval-Augmented Generation (RAG)** para estandarizar respuestas de cobranza utilizando información institucional vigente y trazable.

---

## Regla de negocio para acuerdo de pago

Para este análisis se considera que existe un acuerdo de pago únicamente cuando:

- el cliente manifiesta explícitamente su compromiso de pago;
- existe una fecha específica de pago;

o cuando:

- el asesor propone una fecha concreta;
- el cliente acepta explícitamente dicha fecha.

No se consideran acuerdos:

- propuestas unilaterales del asesor;
- intenciones vagas de pago;
- pagos condicionados;
- fechas de vencimiento;
- pagos ya realizados;
- conversaciones en las que el cliente no confirma el compromiso.

---

## Datos

La base original contiene:

- **42.607 mensajes**
- **1.197 conversaciones únicas**
- 4 tipos de emisor:
  - USUARIO
  - AGENTE
  - BOT
  - HSM

Durante la validación de calidad se identificaron:

- 602 registros completamente duplicados;
- duplicados concentrados en 10 conversaciones;
- equivalencia entre los identificadores de conversación;
- diferencias iniciales entre `total_interacciones` y el número real de mensajes.

Después de eliminar únicamente duplicados exactos:

- se conservaron las 1.197 conversaciones;
- la consistencia de `total_interacciones` alcanzó el 100%.

---

## Metodología

El proyecto se desarrolló mediante las siguientes etapas:

1. Carga y comprensión de los datos.
2. Análisis Exploratorio de Datos (EDA).
3. Validación de calidad.
4. Eliminación de duplicados exactos.
5. Reconstrucción completa de conversaciones.
6. Diseño de taxonomías.
7. Definición de reglas de negocio.
8. Diseño de variables analíticas.
9. Prompt Engineering.
10. Extracción estructurada mediante IA Generativa.
11. Validación mediante Gold Set.
12. Pipeline de procesamiento por lotes.
13. Checkpoints y manejo de errores.
14. Consolidación del dataset analítico.
15. Análisis de preguntas de negocio.
16. Visualización en Power BI.
17. Diseño de propuesta RAG.
18. Generación de insights y recomendaciones.

---

## Variables generadas mediante IA

Cada conversación es transformada en un registro estructurado con variables como:

- `motivo_no_pago`
- `submotivo_no_pago`
- `oferta_asesor`
- `tipo_oferta`
- `argumento_asesor`
- `acuerdo_pago`
- `fecha_acuerdo`
- `sentimiento_cliente`
- `nps_score`
- `satisfaccion_encuesta`
- `facilidad_score`
- `experiencia_ia_score`
- `factores_negativos`
- `resumen_conversacion`
- `recomendacion`

La salida del modelo se valida mediante un esquema estructurado con Pydantic.

---

## Prompt Engineering

El prompt fue diseñado para controlar la interpretación del modelo y reducir errores.

Entre las principales reglas incluidas se encuentran:

- identificar ofertas únicamente a partir de mensajes del AGENTE;
- utilizar BOT y HSM únicamente como contexto;
- no inventar información;
- devolver `No identificado` o `null` cuando no exista evidencia suficiente;
- aplicar estrictamente la definición de acuerdo de pago;
- diferenciar NPS, satisfacción, facilidad y experiencia inferida;
- generar resúmenes breves;
- producir recomendaciones accionables.

---

## Medición de experiencia y satisfacción

Se mantienen separadas las distintas métricas de experiencia:

- `nps_score`: escala explícita de 0 a 10.
- `satisfaccion_encuesta`: escala explícita de 1 a 7.
- `facilidad_score`: escala explícita de 1 a 5.
- `experiencia_ia_score`: indicador inferido mediante IA de 1 a 5.

No se construyó un índice único que combinara estas escalas, debido a que representan dimensiones diferentes de experiencia.

Cuando no existe una evaluación explícita, `experiencia_ia_score` se utiliza como indicador exploratorio de experiencia conversacional.

---

## Validación

Se construyó un Gold Set manual para evaluar las variables más importantes.

Resultados de concordancia piloto:

- Motivo de no pago: **100%**
- Acuerdo de pago: **100%**
- Fecha de acuerdo: **100%**
- Sentimiento: **100%**
- Oferta del asesor: **60%**
- Concordancia global: **92%**

La principal fuente de discrepancia fue el solapamiento semántico entre categorías de oferta.

Por ejemplo:

- refinanciación vs. nueva fecha;
- seguro asociado vs. orientación a otro canal.

Como mejora futura se propone separar:

- **tipo de solución**
- **canal de gestión**

---

## Procesamiento mediante IA

El pipeline fue diseñado para procesar conversaciones por lotes e incluye:

- selección de modelos alternativos;
- reintentos automáticos;
- manejo de errores temporales;
- manejo de límites de cuota;
- checkpoint después de cada conversación procesada;
- recuperación del avance ante interrupciones.

Debido a restricciones de cuota de API durante la prueba técnica, el análisis con IA se ejecutó sobre una **muestra exploratoria de 58 conversaciones**.

Por esta razón, los resultados deben interpretarse como patrones exploratorios y no como estimaciones definitivas para toda la población.

---

## Principales hallazgos

### Motivos de no pago

Dentro de la muestra procesada, uno de los patrones más relevantes fue la presencia de clientes que manifestaban haber realizado previamente el pago o encontrarse al día.

Esto evidencia una posible oportunidad de mejora en la sincronización entre:

- recaudos;
- actualización de cartera;
- campañas de cobranza.

También se identificaron dificultades reales de pago asociadas principalmente a:

- falta de liquidez;
- desempleo o pérdida de ingresos;
- inconformidad con el valor cobrado;
- falta de información o extractos.

---

## Ofertas de los asesores

Las estrategias más frecuentes estuvieron relacionadas con:

- acuerdos o compromisos de pago;
- refinanciación o reestructuración;
- información de saldo;
- orientación a otros canales;
- alternativas de canal de pago;
- nuevas fechas de pago.

---

## Efectividad observada

En la muestra procesada, las acciones más concretas presentaron mayores tasas observadas de acuerdo.

Ejemplos:

- acuerdo o compromiso de pago;
- nueva fecha de pago;
- pago parcial;
- pago total.

Estas tasas deben interpretarse con cautela.

Los resultados muestran **asociaciones observadas**, no relaciones causales.

Adicionalmente, algunas categorías pueden presentar solapamiento conceptual con el resultado final de acuerdo.

---

## Experiencia del cliente

Las conversaciones con menor nivel de experiencia presentaron patrones como:

- falta de resolución;
- pagos no aplicados;
- repetición de información;
- guiones rígidos;
- explicaciones insuficientes;
- transferencias entre áreas;
- fricción multicanal;
- esfuerzo elevado por parte del cliente.

Una oportunidad prioritaria es resolver primero los problemas operativos antes de continuar con presión de cobranza.

---

## Dashboard Power BI

Se desarrolló un dashboard ejecutivo en Power BI que responde a las preguntas principales del análisis.

Incluye:

- conversaciones analizadas;
- acuerdos logrados;
- tasa de acuerdo;
- principales motivos de no pago;
- ofertas realizadas por los asesores;
- tasa de acuerdo por tipo de oferta;
- argumentos observados en conversaciones con acuerdo.

Los resultados corresponden a la muestra procesada de 58 conversaciones.

---

## Propuesta RAG

Se propone una arquitectura basada en Retrieval-Augmented Generation para asistir a los asesores de cobranza.

Flujo conceptual:

Cliente / WhatsApp  
→ Clasificación de intención y motivo  
→ Retrieval  
→ Base vectorial  
→ LLM  
→ Guardrails  
→ Respuesta sugerida al asesor

### Base de conocimiento

La base de conocimiento podría incluir:

- políticas de cobranza;
- ofertas vigentes;
- scripts aprobados;
- condiciones de refinanciación;
- seguros;
- canales de pago;
- preguntas frecuentes;
- procedimientos;
- restricciones regulatorias.

### Evaluación técnica

Métricas propuestas:

- Recall@K
- Precision@K
- MRR
- Faithfulness
- Groundedness
- Answer Relevance
- Tasa de alucinación

### Evaluación de negocio

Indicadores propuestos:

- tasa de acuerdos;
- pago efectivo;
- resolución en primer contacto;
- satisfacción;
- tiempo de gestión;
- escalamiento a humano.

Se recomienda un enfoque **human-in-the-loop** para casos ambiguos o de mayor riesgo.

---

## Recomendaciones de negocio

1. **Sincronizar pagos y campañas de cobranza**  
   Reducir contactos a clientes que ya realizaron el pago.

2. **Diseñar discursos diferenciados por motivo de no pago**  
   Liquidez, desempleo, inconformidad o falta de información requieren estrategias distintas.

3. **Priorizar acciones concretas**  
   Definir fecha, monto, canal y condiciones de manera clara.

4. **Mejorar la resolución en primer contacto**  
   Reducir transferencias y reprocesos.

5. **Implementar RAG como asistente del asesor**  
   Estandarizar respuestas utilizando información institucional vigente.

6. **Monitorear continuamente la experiencia**  
   Utilizar sentimiento, encuestas y factores negativos para coaching y control de calidad.

---

## Metodologías analíticas complementarias

Como evolución futura se proponen metodologías como:

- análisis de sentimiento;
- clustering semántico;
- clasificación de intención;
- análisis de argumentos;
- A/B testing de estrategias de negociación.

Estas técnicas permitirían profundizar en los patrones identificados y validar qué estrategias generan mejores resultados.

---

## Limitaciones

Las principales limitaciones de la prueba son:

- procesamiento de IA realizado sobre una muestra de 58 conversaciones;
- restricciones de cuota de API;
- Gold Set piloto con tamaño limitado;
- solapamiento semántico entre algunas categorías de oferta;
- resultados exploratorios que no deben interpretarse como causalidad.

---

## Escalabilidad

El pipeline fue diseñado para poder extenderse a las 1.197 conversaciones mediante:

- procesamiento por lotes;
- checkpoints;
- reintentos;
- control de errores;
- almacenamiento incremental.

En producción se recomienda:

- ampliar el Gold Set;
- utilizar una única versión de modelo validada;
- monitorear drift;
- validar cambios de modelo antes del despliegue;
- incorporar experimentación controlada;
- medir impacto mediante KPIs de negocio.

---

## Estructura del repositorio

```text
voc-cobranza-ai/
│
├── README.md
│
├── notebooks/
│   └── Prueba_Tecnica_Cientifico_AI_VoC.ipynb
│
├── powerbi/
│   ├── Dashboard_VoC.pbix
│   └── Dashboard_VoC.pdf
│
├── presentation/
│   ├── Presentacion_Final_VoC_Cientifico_AI.pdf
│   
│
└── data/
    └── dataset_voc_powerbi.xlsx

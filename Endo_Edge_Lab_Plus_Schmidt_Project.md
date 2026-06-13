# Endo Edge Lab+
## Plataforma de Inferencia Endocrina Personal mediante Wearables y Edge AI

### Autor
Proyecto conceptual discutido para monitoreo del síndrome de Schmidt (APS-2), diabetes tipo 1 y actividad neuroendocrina.

---

# 1. Motivación

La observación inicial fue que la glucemia presenta un ascenso matutino consistente entre las 04:00 y 08:00 hs durante los días laborales, mientras que durante los fines de semana la curva tiende a aplanarse.

Adicionalmente:

- Diagnóstico de síndrome de Schmidt (APS-2).
- Diabetes tipo 1.
- Tratamiento con hidrocortisona.
- Tratamiento con insulina basal.
- Antecedentes de bilirrubina elevada normalizada tras aumentar la hidrocortisona de 30 mg/día a 40 mg/día.
- Cortisol sérico relativamente elevado en análisis recientes.

Hipótesis inicial:

> La curva glucémica podría contener información indirecta sobre la actividad cortisol-adrenal y el estado de estrés fisiológico.

---

# 2. Evolución de la hipótesis

Inicialmente se planteó:

> Estimar cortisol a partir de la glucemia.

Posteriormente se concluyó que un objetivo más realista es:

> Inferir el estado neuroendocrino funcional mediante datos continuos de sensores.

Debido a que la glucemia está influenciada por múltiples variables:

G = f(C, I, S, H, E, A, St)

Donde:

- C = cortisol efectivo
- I = insulina
- S = sensibilidad a insulina
- H = hidrocortisona
- E = ejercicio
- A = alimentación
- St = estrés

---

# 3. Concepto de Actividad Glucocorticoide Efectiva (AGE)

En lugar de estimar un valor de laboratorio de cortisol:

Se propone modelar:

- Actividad glucocorticoide efectiva.
- Activación simpática.
- Estrés fisiológico.
- Riesgo adrenal.
- Riesgo de hiperglucemia matutina.

AGE representa el efecto fisiológico observado más que una concentración sérica puntual.

---

# 4. Carga de Estrés Glucémica

Se propuso un indicador simple:

SEG = (G7am - G3am) / 4

Unidades:

mg/dL/h

Interpretación tentativa:

- < 3: baja activación
- 3-8: moderada
- 8-15: alta
- > 15: muy alta

El índice podría correlacionarse con:

- Estrés laboral
- Calidad de sueño
- Activación simpática
- Hidrocortisona efectiva

---

# 5. Integración con frecuencia cardíaca

La frecuencia cardíaca aporta información sobre activación simpática.

Modelo conceptual:

SEG = α(dG/dt) + β(ΔFC)

Donde:

- dG/dt = pendiente glucémica
- ΔFC = variación de frecuencia cardíaca

Posteriormente se propuso incluir HRV.

---

# 6. Índice Neuroendocrino Matutino (INEM)

Modelo conceptual:

INEM = w1(ΔG) + w2(ΔFC) - w3(HRV)

Variables:

- ΔG = variación glucémica
- ΔFC = incremento frecuencia cardíaca
- HRV = variabilidad cardíaca

Interpretación:

Valores elevados podrían representar:

- Estrés fisiológico
- Activación simpática
- Aumento de riesgo hiperglucémico
- Posible necesidad de revisión terapéutica

---

# 7. Gemelo Digital Endocrino

Objetivo final:

Construir un modelo personalizado del paciente.

Variables observadas:

- CGM
- Frecuencia cardíaca
- HRV
- Sueño
- Temperatura
- Actividad física
- Hidrocortisona
- Insulina
- Laboratorio

Variables inferidas:

- Activación neuroendocrina
- Actividad glucocorticoide efectiva
- Estrés fisiológico
- Riesgo adrenal
- Riesgo glucémico

---

# 8. Reorientación de Endo Edge Lab+

Objetivo original:

Detección de enfermedades crónicas mediante wearables durante 3 meses.

Nueva propuesta:

Inferencia endocrina personalizada para síndrome de Schmidt.

Ventajas:

- Problema clínico más acotado.
- Biomarcadores disponibles.
- Posibilidad de estudios longitudinales N=1.
- Mayor factibilidad de validación.

---

# 9. Arquitectura propuesta

Sensores:

- FreeStyle Libre 2
- Smartwatch
- Registro farmacológico

Datos capturados:

CGM:
- Glucemia
- Tendencia
- Variabilidad

Wearable:
- FC
- HRV
- Sueño
- Temperatura
- Actividad

Medicaciones:
- Hidrocortisona
- Glargina
- Bolos

---

# 10. Edge AI con MAX78002

Hardware principal:

MAX78002

Funciones:

- Preprocesamiento local
- Extracción de características
- Inferencia en tiempo real
- Bajo consumo

Características calculadas:

- Pendiente glucémica
- Variabilidad glucémica
- FC promedio
- HRV
- Calidad de sueño
- Índices endocrinos

---

# 11. Comunicación con smartphone

Se concluyó que la mejor arquitectura inicial es:

FreeStyle Libre 2 -> Juggluco/xDrip+ -> Smartphone -> BLE -> MAX78002

Ventajas:

- Evita ingeniería inversa compleja.
- Aprovecha ecosistema existente.
- Mayor robustez.
- Menor costo de desarrollo.

---

# 12. Bluetooth recomendado

Se analizaron varias opciones.

Recomendación principal:

nRF52840

Motivos:

- BLE 5.x
- Excelente SDK
- Amplia adopción en wearables médicos
- OTA
- Bajo consumo

Arquitectura:

MAX78002 <-> UART/SPI <-> nRF52840 <-> Smartphone

Alternativas:

- ESP32-C3
- DA14531
- HM-19
- AT-09
- JDY-25

---

# 13. Frameworks para MAX78002

SDK:

- Analog Devices MSDK

IA:

- ai8x-training

Capacidades:

- Entrenamiento en PyTorch
- Cuantización
- Exportación al acelerador CNN del MAX78002

---

# 14. Roadmap de investigación

## Fase 1

Correlación:

- Glucemia
- FC
- HRV
- Sueño

Objetivo:

Detectar patrones de estrés fisiológico.

## Fase 2

Agregar:

- Cortisol
- ACTH
- Hidrocortisona

Objetivo:

Estimar AGE.

## Fase 3

Migrar inferencia al MAX78002.

Objetivo:

Sistema Edge AI autónomo.

## Fase 4

Gemelo Digital Endocrino.

Objetivo:

Modelo personalizado predictivo.

---

# 15. Hipótesis principal

Las señales combinadas de:

- CGM
- Frecuencia cardíaca
- HRV
- Sueño
- Temperatura
- Registro farmacológico

permiten inferir el estado neuroendocrino funcional en pacientes con síndrome de Schmidt con precisión clínicamente útil mediante Edge AI.

---

# 16. Resultado esperado

El sistema no pretende medir cortisol directamente.

El sistema pretende estimar:

- Actividad Glucocorticoide Efectiva (AGE)
- Índice Neuroendocrino Matutino (INEM)
- Estado de Estrés Fisiológico
- Riesgo de Hiperglucemia Matutina
- Riesgo Adrenal

constituyendo un Gemelo Digital Endocrino personalizado.

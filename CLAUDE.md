# Endo-Edge LAB+ — Sistema de Monitorización Endocrina Personal

## Descripción

Sistema ciberfísico de inferencia endocrina personalizada para monitorización continua del Síndrome de Schmidt (APS-2 / Síndrome Pluriglandular Tipo 2). Integra datos de biosensores continuos (CGM, wearables) con registros farmacológicos y analítica de laboratorio para predecir hipoglucemia, estimar actividad glucocorticoide efectiva y detectar estrés fisiológico mediante Edge AI. Diseño N=1 longitudinal.

## Estado actual

**Implementación: 0%** — Todo el código en `endoedge-node1/` son stubs vacíos.
La especificación completa está en `Documentación Técnica/PROMPT_MAESTRO_EndoEdge_LAB+.txt` (32 KB).
Las deudas técnicas críticas están en `deuda_tecnica/DT-Master.md` (54 ítems).

**Leer PROMPT_MAESTRO y DT-Master antes de cualquier implementación.**

## Arquitectura de 3 nodos

```text
Node 1 (Wearable)            Node 2 (On-Premise Server)     Node 3 (Lab)
MAX78002 + nRF52840          Python/FastAPI + HAPI FHIR R4   HL7 FHIR R4
LSTM INT8 <3ms               LSTM + Delphi-2M                16 analitos clínicos
DSS edge                     DSS refinado + RTM + Catálogo   LOINC / ICD-10
      ↓ BLE/UART                       ↑                           ↑
      └─────────── TLS 1.3 ────────────┘───────── FHIR API ────────┘
```

## Outputs del sistema

| Output | Descripción | Umbral |
| ------ | ----------- | ------ |
| **DSS** (Symbiotic Drift Score) | 0-100, desviación del baseline personal | >76 sostenido 6h → evaluación clínica |
| **RTM** (Risk Trend Marker) | 5 zonas de estratificación temporal | pts/semana de velocidad |
| **Catálogo LAB** | 10 probabilidades diferenciales Fase I | AUROC target ≥0.88 |

## Hardware Node 1

- **MCU principal**: Analog Devices MAX78002 (Cortex-M4F 120 MHz + RISC-V 60 MHz + CNN Accelerator 200 MHz)
- **Coprocesador comunicaciones**: Seeed XIAO nRF52840 (antena chip integrada, BLE 5.0, UART → MAX78002)
- **CGM**: FreeStyle Libre 2 → Juggluco (Android) → BLE → Seeed XIAO nRF52840 → UART → MAX78002
- **ECG / HRV / FC**: Polar H10 (chest strap, 130 Hz, ECG raw µV) → BLE → Seeed XIAO nRF52840
- **Temperatura continua**: Termistor NTC externo → ADC MAX78002 (directo)
- **IMU**: Acelerómetro/giroscopio → SPI MAX78002 (directo)

### Ruta de adquisición por fases

**Fase 1 (actual):**

- CGM: FreeStyle Libre 2 -> Juggluco -> Smartphone -> Seeed XIAO nRF52840 -> UART -> MAX78002
- Cardiaco: Polar H10 -> BLE -> Seeed XIAO nRF52840 -> UART -> MAX78002
- Temperatura: Termistor NTC -> ADC MAX78002 (directo)
- Edge: MAX78002 en modo correlacion y recoleccion de datos

**Fase 3 (objetivo):**

- CGM: FreeStyle Libre 2 NFC directo -> MAX78002 (experimental, ingenieria inversa cifrado)
- Cardiaco: idem Fase 1
- Temperatura: idem Fase 1
- Edge: MAX78002 inferencia autonoma sin smartphone

## Modelos de IA

| Modelo | Donde | Specs | Latencia |
| ------ | ----- | ----- | -------- |
| LSTM bidirecional (edge) | MAX78002 CNN Accel. | ~150K params INT8, [B, 12, 7] | <3 ms |
| LSTM bidirecional (server) | Node 2 / ONNX Runtime | [B, 288, 12] 24h window | — |
| Delphi-2M | Node 2 (server-side) | UK Biobank 0.4M, >1000 enfermedades | — |

Pipeline de conversión: PyTorch → ONNX INT8 → ai8xize.py (MAX78002 CNN format)

## Stack por nodo

| Nodo | Stack |
| ---- | ----- |
| Node 1 firmware | C/FreeRTOS · CMake · Analog Devices MSDK v1.3 |
| Node 1 comms | Rust · embassy-rs · BLE 5.2 · rustls (TLS 1.3) |
| Node 1 ML | PyTorch → ONNX 1.17 → TFLite → ai8xize.py |
| Node 2 server | Python 3.11 · FastAPI · HAPI FHIR R4 (Java 17) · ONNX Runtime 1.17 |
| Node 3 lab | HL7 FHIR R4 · LOINC · ICD-10 |

## Marco de investigación

- **Diseño**: Estudio longitudinal N=1, monitorización continua personal
- **Condición**: Síndrome de Schmidt (APS-2) — Diabetes tipo 1 + Insuficiencia adrenal primaria
- **PI**: Juan Pablo Chancay (Aural-Syncro) — investigador y paciente
- **Normas**: ISO 14155:2020, IEC 62304 (Clase B), Ley 26.529, 25.326, 25.506
- **IP**: PI-001 v1.3 DNDA (Marzo 2026) — Chancay 100% software/hardware/algoritmos

## Wiki de Conocimiento

Wiki personal en: `c:\Users\Usuario\Documents\Aural Syncro\Obsidian`

**Consultar el wiki cuando:**

- Necesites comparar opciones de hardware edge (MAX78002 vs alternativas como Jetson Nano)
- Evalúes modelos de cuantización INT8 para el pipeline PyTorch → MAX78002
- Busques contexto sobre TOPS/W y eficiencia de inferencia en dispositivos médicos

**Actualizar el wiki cuando:**

- Obtengas benchmarks reales de latencia en MAX78002 (contrastar con [[tops-metrica]])
- Documentes el pipeline de cuantización PyTorch → ONNX → MAX78002 como referencia
- Completes la integración de Delphi-2M — crear `wiki/entities/delphi-2m.md`
- Página a actualizar: `wiki/proyectos/endo-edge-lab.md`

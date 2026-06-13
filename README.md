# Endo-Edge LAB+

Sistema ciberfísico de diagnóstico temprano y seguimiento continuo de enfermedades crónicas

Endo-Edge LAB+ combina biosensores wearable continuos con inferencia de inteligencia artificial en el borde (Edge AI) para mapear en tiempo real la interacción dinámica entre vías hormonales, metabólicas y celulares — lo que el proyecto denomina "simbiosis endocrina celular" — con el objetivo de detectar firmas patológicas años antes de su manifestación clínica tradicional.

El sistema no emite diagnósticos médicos definitivos. Genera una señal de alerta preclínica (DSS > 76 sostenido por 6 horas) que justifica evaluación clínica profesional.

Arquitectura de tres nodos
Nodo 1 — Wearable Edge AI (paciente)
MCU principal Analog Devices MAX78002 (ARM Cortex-M4F + RISC-V + CNN Accelerator) con coprocesador de comunicaciones Nordic nRF52840. Captura ECG a 256 Hz, glucosa intersticial (Dexcom G7), conductancia de piel (Empatica EmbracePlus), lactato (PKvitality K'Watch) e iones en sudor (Epicore Discovery Patch) mediante una red BAN de micro-nodos relé sobre textiles conductores. Corre un modelo LSTM bidireccional INT8 (~150K parámetros, latencia < 3 ms) y transmite paquetes firmados RSA-2048 via BLE → TLS 1.3.

Nodo 2 — Inferencial multimodal on-premise
Servidor físico en CABA con stack Python/FastAPI + HAPI FHIR R4 (Java 17) + ONNX Runtime. Motor de Fusión Temporal sobre tensor [B, 288, 12] (24 horas, paso de 5 min, 12 características). Integra Delphi-2M (modelo tipo GPT entrenado en UK Biobank, 0.4M participantes) con la historia clínica ICD-10 del paciente y el DSS continuo del Nodo 1 para predicción de más de 1000 enfermedades. Los datos identificables nunca salen del servidor.

Nodo 3 — Edge de laboratorio
Ingesta de resultados LAB en HL7 FHIR R4 desde LIS de laboratorios (API directa / SFTP cifrado / OCR fallback). Panel de 16 analitos con códigos LOINC.

Outputs principales
Output Descripción
DSS Symbiotic Drift Score (0–100): desviación del ecosistema endocrino del patrón personal aprendido
RTM Risk Trend Marker (5 zonas): tendencia temporal del DSS en pts/semana
Catálogo LAB Fase I Probabilidades diferenciales para 10 patologías metabólicas/endocrinas
Objetivos de rendimiento: AUROC ≥ 0.88 (LAB), ≥ 0.85 (WEAR), ≥ 0.90 (integrado).

Marco de investigación
Diseño: estudio longitudinal N=1, monitorización continua personal
Síndrome de Schmidt (APS-2) — Diabetes tipo 1 — Insuficiencia adrenal primaria
Marco regulatorio: Ley 26.529, Ley 25.326, ISO 14155:2020, IEC 62304, HL7 FHIR R4
Stack tecnológico

Firmware (MAX78002)   C/C++ · FreeRTOS · ONNX INT8 · MSDK Analog Devices
Comunicaciones        Rust · embassy-rs · BLE 5.2 · TLS 1.3 · rustls
ML / Conversión       Python · PyTorch · ONNX · TFLite Micro · ai8xize.py
Backend Nodo 2        Python 3.11 · FastAPI · HAPI FHIR R4 · Java 17
Interoperabilidad     HL7 FHIR R4 · LOINC · ICD-10 · MessagePack
Titularidad intelectual
Juan Pablo Chancay (Aural-Syncro) — IP, arquitectura, firmware, algoritmos (PI-001 v1.3, DNDA Marzo 2026)

Contacto: <juanpablo.chancay@aural-syncro.com.ar> - <www.aural-syncro.com.ar>

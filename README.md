# Centinela del Café ☕🍃

Detección temprana de hojas de café robusta afectadas (roya o ácaro rojo) a partir de una foto. Es el proyecto del curso de Deep Learning (Maestría en Ciencia de Datos, Universidad Santo Tomás).

- **Equipo:** Briyith Lisdey Herrera Cruz · Juan Pablo Becerra Tirano · Johnatan Velez Martinez
- **Cliente:** técnico de campo de un lote de café robusta (CIIDEA‑ESPAM MFL, Calceta, Manabí, Ecuador)
- **Evento:** `hoja afectada (ácaro o roya 1–4) = 1` / `sana = 0`

| Fase | Modelo | Modalidades |
|---|---|---|
| 1 | MLP (2 capas ocultas, salida logit) | Imagen aplanada en 24 descriptores de la hoja |
| 2 | CNN + LSTM/GRU con fusión | Imagen + serie diaria de clima de NASA POWER (60 días antes de la toma) |

## Estructura

```
propuesta/Propuesta_Datos.md         Propuesta de datos v2 (anexos A, B y C)
scripts/01_descarga_verificacion.py  Descarga de Mendeley + SHA-256 de cada archivo
scripts/02_verificar_clave.py        EXIF, tabla día × clase, plausibilidad, pHash, NASA POWER, anexo A
scripts/03_extraer_descriptores.py   Descriptores de la hoja (entrada del MLP)
src/features.py                      Aplanamiento de la imagen (solo dentro del polígono de la hoja)
data/manifest_mendeley.csv           Huellas SHA-256 publicadas y calculadas
data/processed/                      Derivados SIN datos personales (descriptores, metadatos, clima)
resultados/                          Tablas de verificación, anexo A y figuras
fase1/Centinela_Cafe_Fase1.ipynb     Cuaderno de la Fase 1 (se ejecuta completo con "Ejecutar todo")
fase1/Informe_Fase1.pdf              Informe técnico-ético (≤ 4 páginas)
fase1/Bitacora_IA.md                 Bitácora de uso de IA generativa
tools/                               Generadores del cuaderno y del informe
```

`data/raw/` (fotos y anotaciones originales) **no se versiona**: contiene los correos de los anotadores. Para reconstruirla, ejecute los scripts.

## Reproducir

```bash
pip install -r requirements.txt
python scripts/01_descarga_verificacion.py   # ~2,2 GB, verifica huellas
python scripts/02_verificar_clave.py
python scripts/03_extraer_descriptores.py
jupyter nbconvert --to notebook --execute fase1/Centinela_Cafe_Fase1.ipynb
```

En Colab basta con abrir `fase1/Centinela_Cafe_Fase1.ipynb`. Si no encuentra `data/processed/descriptores_rocole.csv`, lo descarga de este repositorio: ajuste `URL_REPO_RAW` en la primera celda.

## Datos y licencias

- **RoCoLe v2:** Parraga‑Alava, J., Cusme, K., Loor, A., & Santander, E. (2019). Mendeley Data. https://doi.org/10.17632/c5yvn32dzg.2 — **CC BY 4.0**. Los derivados de `data/processed/` se redistribuyen con esta atribución. Cambios realizados: se eliminó el correo de los anotadores, se calcularon descriptores y se excluyeron duplicados.
- **NASA POWER:** NASA Langley Research Center, POWER Project. https://power.larc.nasa.gov

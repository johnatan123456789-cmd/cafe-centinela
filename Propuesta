# Centinela del Café — Propuesta de Datos (v2, 2026-11-04)

| | |
|---|---|
| **Curso** | Deep Learning — Maestría en Ciencia de Datos, Universidad Santo Tomás |
| **Equipo** | Briyith Lisdey ______________________ · Juan Pablo ______________________ · Johnatan ______________________ |
| **Firmas** | ______________________ · ______________________ · ______________________ |
| **Versión** | v2 — cambia la clave de `(Calceta, mes)` a `(Calceta, día de toma)` y añade la tabla día × clase. Ver el registro de cambios al final |

---

## 1. Evento de riesgo, cliente e impacto

- **Cliente:** el técnico de campo de un lote de café robusta, como el del CIIDEA‑ESPAM MFL (Calceta, Manabí, Ecuador).
- **Pregunta:** a partir de una foto de una hoja, ¿la planta está afectada y hay que inspeccionarla?
- **Impacto:** la roya (*Hemileia vastatrix*) defolia la planta y reduce la cosecha del ciclo siguiente. Si se detecta tarde, las esporas pasan a las plantas vecinas con el viento y la lluvia. El ácaro rojo (*Oligonychus* spp.) broncea la hoja y reduce la fotosíntesis.
- **Objetivo binario:** `hoja afectada = 1` si tiene ácaro rojo o roya de cualquier nivel (1 a 4); `sana = 0`. Para el técnico, cualquier hoja afectada exige inspección: la severidad no cambia la acción inmediata. **Lo que se pierde:** la severidad. Se conserva como variable para analizar errores por subgrupo (sensibilidad por ácaro, roya 1, 2, 3 y 4).
- **Balance:** 769 afectadas frente a 791 sanas (49,3 %). Línea base de clase mayoritaria ≈ 50,7 %.

## 2. Fuentes (condiciones a y b)

| Modalidad | Fuente | Licencia | Rol |
|---|---|---|---|
| Imagen | Parraga‑Alava, J., Cusme, K., Loor, A., & Santander, E. (2019). *RoCoLe: A robusta coffee leaf images dataset* (v2) [Data set]. Mendeley Data. https://doi.org/10.17632/c5yvn32dzg.2. Artículo de datos: *Data in Brief*, 25, 104414. https://doi.org/10.1016/j.dib.2019.104414 | CC BY 4.0 | Fase 1: aplanada en 24 descriptores. Fase 2: CNN |
| Serie de tiempo | NASA POWER, API diaria (comunidad AG): T2M, T2M_MAX, T2M_MIN, RH2M, PRECTOTCORR, WS2M, ALLSKY_SFC_SW_DWN. https://power.larc.nasa.gov | Datos públicos de la NASA (uso libre con atribución) | Fase 2: LSTM/GRU |

**¿La imagen contiene algo que un número no?** Sí. La forma, la ubicación y la textura de las pústulas y del bronceado se ven en la hoja y no aparecen en ningún registro de clima.

## 3. Clave de alineación (condición d)

**Lugar.** Lo documenta el artículo de datos (tabla de especificaciones): CIIDEA, ESPAM MFL, Calceta, Manabí, lat −0,834206, lon −80,177159. Es un campo experimental institucional, no una finca privada. **El EXIF no trae GPS (0 de 1 560)**, así que el lugar sale del artículo y no de las fotos.

**Fecha.** Se verificó en **las 1 560 fotos** (no en una muestra) con `scripts/02_verificar_clave.py`. El 100 % trae `DateTimeOriginal`, y siempre coincide con `DateTime` (IFD0).

**Clave v2:** `(Calceta, día de toma)`. El vector de clima es la ventana de **60 días que termina el día anterior** a la toma, consultada en NASA POWER en las coordenadas del artículo.

### 3.1 Días distintos y fotos por clase y día

Hay **5 días distintos** de toma. Archivos completos: `resultados/tabla_dia_x_clase_*.csv`.

| Día de toma | Sanas | Afectadas | % afectadas | Ácaro | Roya 1 | Roya 2 | Roya 3 | Roya 4 | Teléfono(s) |
|---|---|---|---|---|---|---|---|---|---|
| 2019‑04‑09 | 45 | 47 | 51,1 | 4 | 20 | 12 | 5 | 6 | SM‑J200M |
| 2019‑04‑10 | 214 | 208 | 49,3 | 41 | 92 | 53 | 15 | 7 | SM‑J200M, SM‑J700M |
| 2019‑04‑11 | 391 | 375 | 49,0 | 112 | 152 | 62 | 33 | 16 | SM‑G610M, SM‑J200M, SM‑J700M |
| 2019‑05‑01 | 70 | 62 | 47,0 | 9 | 43 | 7 | 2 | 1 | SM‑J200M |
| 2019‑05‑02 | 71 | 77 | 52,0 | 1 | 37 | 32 | 7 | 0 | SM‑J320H |
| **Total** | **791** | **769** | **49,3** | 167 | 344 | 166 | 62 | 30 | 4 modelos Samsung |

**¿La fecha separa las clases? No.** Cada día tiene entre 47 % y 52 % de hojas afectadas.
- La V de Cramér día~clase es **0,024** (0,033 sin duplicados).
- Predecir con la clase mayoritaria de cada día da una exactitud de **0,512**, frente a 0,507 de la clase mayoritaria global.
- En el cuaderno, la línea base «solo el día» obtiene F1 = 0,28 en prueba.

Sanas y afectadas se fotografiaron en las mismas jornadas. La sospecha de que vinieran de sesiones distintas por estar en archivos separados no se cumple aquí: en RoCoLe las clases están en una hoja de cálculo, no en carpetas.

**El teléfono sí depende del día** (V de Cramér día~teléfono = 0,694), pero no de la clase (V = 0,028). Mitigaciones:
- Los descriptores se calculan solo dentro del polígono de la hoja, sin el fondo.
- El cuaderno hace una prueba *leave‑one‑day‑out*: F1 entre 0,67 y 0,83 frente a 0,79 en la partición por planta, sin caída sistemática.

### 3.2 Plausibilidad de las fechas

El nombre del archivo no trae fecha. Por eso no se puede contrastar contra el EXIF; las fechas se contrastan contra hechos externos al reloj de la cámara:

| Revisión | Resultado |
|---|---|
| Valores por defecto (1970, 1980, 2000, 01‑01, 00:00:00) | 0 |
| Posteriores al inicio de la anotación en Labelbox (`Created At` mínimo = 2019‑05‑14) | 0 |
| Posteriores a la publicación en Mendeley (2019‑05‑17) | 0 |
| Toma posterior a su propia anotación | 0 |
| Hora de toma | Todas entre 09:01 y 12:33, horario de trabajo de campo de mañana |
| Orden coherente | Los bloques `C` se recorrieron en secuencia (C9 y C12 el 9 de abril → C6, C7, C10, C11 el 10 → C2, C3, C4, C8 el 11 → C5 el 1 de mayo → C1 el 2 de mayo). Solo 5 de 390 plantas tienen fotos en dos días seguidos |
| Coherencia con el artículo | El artículo dice que las fotos se tomaron *«daily under real conditions»*; las fechas caen en un único periodo de 3,5 semanas antes de la anotación |

**Limitación declarada:** las cuatro cámaras pudieron tener el reloj desfasado algunas horas o días. Con una ventana de 60 días, un desfase de ±1 día cambia muy poco el vector de clima: la precipitación acumulada varía menos del 3 % entre días consecutivos.

### 3.3 ¿Aplica la ventana de 60 días a la roya del café?

Sí, y el argumento biológico es parecido al de la monilia:

- *H. vastatrix* necesita agua libre en la hoja durante varias horas para germinar. Su temperatura óptima de infección es de unos 21–25 °C.
- El **periodo de latencia** (de la infección a la lesión esporulante visible) dura unas 3–5 semanas según la temperatura (Kushalappa & Eskes, 1989). En Calceta la media de los 60 días previos fue de ≈ 25,7 °C, así que la latencia está en el extremo corto.
- Una pústula visible el día *d* refleja condiciones de infección entre *d* − 60 y *d* − 15, aproximadamente. Por eso la ventana debe **terminar el día anterior** (sin información del día de la foto) y cubrir al menos un ciclo de latencia con margen.
- El **ácaro rojo** prospera con tiempo seco. La misma ventana capta la falta de lluvia.

Para la Fase 2, la LSTM recibe la serie diaria completa (60 × 7). Como análisis interpretable se agregan sub‑ventanas: [d‑60, d‑36], [d‑35, d‑15] y [d‑14, d‑1].

### 3.4 Lo que la clave v2 no resuelve (aviso del docente)

Con 5 días hay 5 vectores de clima, muy parecidos entre sí: 643,8 / 641,9 / 627,3 / 518,4 / 517,8 mm de lluvia acumulada; la temperatura media varía 0,14 °C y la humedad relativa media 0,4 puntos. La prevalencia de hojas afectadas es igual todos los días, así que **la rama temporal casi no puede aportar a la clasificación de la hoja**. Ver el §7.

## 4. Datos que requieren EDA (condición c): huella de crudeza

| Hallazgo | Evidencia | Tratamiento |
|---|---|---|
| Celdas vacías en el CSV de Labelbox | **22,2 %** (4 de 18 columnas casi vacías: `Agreement`, `Benchmark *`) | Se eliminan |
| Correo del anotador | Columna `Created By` (4 correos personales) | **Se borra** al cargar y se reemplaza por un hash (`anot_xxxxxx`) |
| Tres resoluciones | 2048×1152 (768), 1280×720 (479), 4128×2322 (313), según el teléfono | Se reescala el lado mayor a 512 px |
| Orientación EXIF | 6 y 8 (rotadas) en la mayoría | Se verificó que el polígono está en el marco crudo |
| Fondo heterogéneo | Maleza, suelo, cielo, manos | Descriptores solo dentro del polígono de la hoja |
| **Duplicados exactos** | **67 fotos repetidas byte a byte (mismo SHA‑256) con nombres de plantas distintas** (134 archivos). En 27 la severidad difiere entre copias y en 1 la etiqueta binaria se contradice | Se deja una copia; se excluye la foto con etiqueta binaria contradictoria. Quedan 1 492 fotos únicas. `resultados/duplicados_exactos.csv` |
| Casi duplicados | 92 pares con pHash ≤ 6 bits de Hamming, todos entre plantas distintas | Las plantas unidas por pHash forman un **grupo de partición**: 336 grupos para 390 plantas; el mayor tiene 21 plantas |
| Nombre fuera de convención | `C11P11HE.jpg` (el patrón es `C#P#[H\|E]#`) | Se asigna a la planta C11P11 y se registra |
| Severidad muy desbalanceada | Roya 3 = 62 y roya 4 = 30 fotos | Se agrupa en binaria; la severidad se usa solo en el análisis de errores |

Convención de nombres: `C` = bloque del lote, `P` = planta, `H`/`E` = haz/envés y el dígito = hoja 1 o 2. Son 4 fotos por planta.

## 5. Partición

`StratifiedGroupKFold` (7 pliegues, semilla 42) con **grupo = componente de plantas unidas por pHash**: 5 pliegues para entrenamiento (1 117 fotos), 1 para validación (160) y 1 para prueba (215). El cuaderno comprueba que ningún grupo aparece en dos conjuntos. Los cinco días quedan representados en los tres conjuntos.

La partición cronológica no aplica en la Fase 1 porque la modalidad aplanada es la imagen, no la serie. En la Fase 2 se reportará además la partición por día (*leave‑one‑day‑out*), ya implementada en el cuaderno.

## 6. Riesgos éticos (condición e)

| Riesgo | Mitigación |
|---|---|
| Falso negativo: una hoja con roya o ácaro declarada sana, y el foco se extiende | Umbral elegido en validación con costo FN = 5 × FP; se reporta la sensibilidad por severidad y por día |
| Sesgo de dispositivo: el modelo aprende el teléfono o la iluminación del día | Descriptores dentro de la hoja; prueba leave‑one‑day‑out; exactitud por teléfono |
| Generalización a otros lotes | Todos los datos vienen de un solo lote de robusta en la costa ecuatoriana. Se declara que el modelo no está validado para arábica ni para otras zonas |
| Datos personales | Correo del anotador eliminado; manos visibles en algunas fotos (sin rostros); sin GPS |
| Uso de IA generativa | Solo se le muestran esquemas y filas sintéticas o anonimizadas (ver la bitácora) |

## 7. Plan para la Fase 2 (respuesta al punto 2 del docente)

1. **Ablación pre‑registrada:** imagen sola (CNN), clima solo (LSTM sobre los 60 días) y fusión. La hipótesis declarada desde ahora es que **clima solo ≈ clase mayoritaria y fusión ≈ imagen sola**, porque la prevalencia no cambia entre días. Si se confirma, se reporta como **hallazgo honesto**: con un solo lote y 5 días, el clima no informa sobre qué hoja está enferma. Esa es una conclusión útil para el cliente.
2. **Pregunta temporal que sí varía (alternativa, para consultar con el docente):** usar la rama temporal para estimar el **riesgo de condiciones favorables a la roya** en la ventana previa (por ejemplo, días con RH ≥ 85 % y 21–25 °C) como contexto para la decisión de inspección, no como predictor de la hoja.
3. **Ganar variación con otra fuente (opcional):** una segunda fuente de imágenes de café con fecha y lugar documentados. Solo se incorporaría con aprobación previa y con la misma verificación de este documento (EXIF, tabla día × clase y plausibilidad).

---

## Anexo A — 10 filas reales de acoplamiento

Generado por `scripts/02_verificar_clave.py` → `resultados/anexo_A_10_filas.csv`. Dos fotos por día, sin duplicados. La huella es el SHA‑256 calculado; «=Mendeley» indica que coincide con la huella que publica la API.

| Clave | Archivo | SHA‑256 (12) | =Mendeley | Toma (EXIF) | Teléfono | Clase | Ventana | Precipitación 60 d (mm) | T2M (°C) | RH2M (%) | Días con RH ≥ 85 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| (Calceta, 2019‑04‑09) | C9P11H2.jpg | dfa0daaf970e | n/d¹ | 09:48:05 | SM‑J200M | sana | 02‑08…04‑08 | 643,8 | 25,80 | 85,8 | 37 |
| (Calceta, 2019‑04‑09) | C9P7H1.jpg | 2ec226049a82 | n/d¹ | 09:40:31 | SM‑J200M | roya 1 | 02‑08…04‑08 | 643,8 | 25,80 | 85,8 | 37 |
| (Calceta, 2019‑04‑10) | C6P32H1.jpg | f26c3169f99d | n/d¹ | 09:37:03 | SM‑J200M | roya 1 | 02‑09…04‑09 | 641,9 | 25,79 | 85,9 | 37 |
| (Calceta, 2019‑04‑10) | C7P20E2.jpg | bc63cff2b15c | n/d¹ | 10:23:08 | SM‑J200M | roya 2 | 02‑09…04‑09 | 641,9 | 25,79 | 85,9 | 37 |
| (Calceta, 2019‑04‑11) | C3P37E2.jpg | b43c94dd5488 | sí | 10:52:01 | SM‑G610M | ácaro rojo | 02‑10…04‑10 | 627,3 | 25,79 | 85,8 | 36 |
| (Calceta, 2019‑04‑11) | C10P38E2.jpg | e0ec264762a6 | sí | 10:05:16 | SM‑J700M | roya 1 | 02‑10…04‑10 | 627,3 | 25,79 | 85,8 | 36 |
| (Calceta, 2019‑05‑01) | C5P18H1.jpg | b691f0942112 | sí | 11:53:33 | SM‑J200M | sana | 03‑02…04‑30 | 518,4 | 25,67 | 86,2 | 42 |
| (Calceta, 2019‑05‑01) | C5P26E2.jpg | d6847ef7549e | sí | 12:05:01 | SM‑J200M | roya 2 | 03‑02…04‑30 | 518,4 | 25,67 | 86,2 | 42 |
| (Calceta, 2019‑05‑02) | C1P25E2.jpg | def8c0381d49 | sí | 10:43:20 | SM‑J320H | roya 1 | 03‑03…05‑01 | 517,8 | 25,66 | 86,2 | 43 |
| (Calceta, 2019‑05‑02) | C1P10E1.jpg | d850f6078e79 | sí | 10:16:11 | SM‑J320H | sana | 03‑03…05‑01 | 517,8 | 25,66 | 86,2 | 43 |

¹ La API pública de Mendeley lista como máximo 1 000 archivos por carpeta, así que 556 fotos no tienen huella publicada para contrastar. Su huella calculada sobre el ZIP oficial queda en `data/manifest_mendeley.csv`.

## Anexo B — Cadena de procedencia

| Elemento | Valor |
|---|---|
| Fuente original | Mendeley Data, RoCoLe **v2** (publicada el 2019‑05‑17), DOI 10.17632/c5yvn32dzg.2, CC BY 4.0 |
| Fecha de descarga | 2026‑11‑04 (UTC), con `scripts/01_descarga_verificacion.py` |
| Método | ZIP oficial (`https://data.mendeley.com/public-api/zip/c5yvn32dzg/download/2`) + listado de la API pública con el SHA‑256 de cada archivo |
| SHA‑256 del ZIP oficial (2 245 588 288 bytes) | `31daa765fb0cc27d4b9b897fa6350aacf0087ee5771d3c7599d2ac7041b866ff` |
| SHA‑256 `RoCoLE-csv.csv` | `a9aa6b411569dba7e9ba56fed79e5cd82af04a0ef092e56309158d632ac5205b` (= Mendeley) |
| SHA‑256 `RoCoLe-classes.xlsx` | `0040a57ffd441bd69ead997309d14c8d7f6fdfc83c5dbdc47805e0bc4da05ef2` (= Mendeley) |
| SHA‑256 `RoCoLE-json.json` | `286b73725b7c027a6bc9a1118a281b026e1ec9e35c3510398ed6bbc0e579cf99` (= Mendeley) |
| SHA‑256 `RoCoLE-coco.json` | `beac06144c2865cd0ceb2ef3f81791334cf1674400184ff1aa123d1e67f7ec4f` (= Mendeley) |
| Fotos | 1 560 extraídas; **1 004 archivos contrastados con la huella publicada: 0 diferencias**; 556 solo con huella calculada (ver nota del anexo A) |
| SHA‑256 del manifiesto (`data/manifest_mendeley.csv`) | `88fc9bc6488c78e20954995989e9ca6a99fe2a5abb1af3a480c7a33d0f326af9` |
| Huella de crudeza | 22,2 % de celdas vacías; correos de anotador; 3 resoluciones; 67 duplicados exactos con nombres distintos (27 con severidad en conflicto); `C11P11HE.jpg` fuera de convención. Una réplica «limpia» no tendría estos rasgos |
| Réplicas encontradas (búsqueda web, 2026‑11‑04) | Roboflow Universe (`coffee-leaf-detection/rocole-xot5m`, `rocole-segment`), Hugging Face (`bhugxer/RoCoLe-Coffee`), Dataset Ninja (`rocole`), gts.ai. **No se usan**: solo se usa el original de Mendeley |
| Búsqueda en Kaggle | **Pendiente del equipo:** buscar «RoCoLe» y «robusta coffee leaf» en kaggle.com/datasets y registrar aquí la fecha, la captura de pantalla y las réplicas encontradas, contrastando su tamaño y estructura con la huella de Mendeley |
| NASA POWER | API diaria, punto (−0,834206, −80,177159), 2019‑02‑07 → 2019‑05‑02, consultada el 2026‑11‑04 → `data/processed/nasa_power_calceta_diario.csv` |

## Anexo C — Checklist de protección de datos (Ley 1581 de 2012), por fuente

| Ítem | RoCoLe | NASA POWER |
|---|---|---|
| ¿Contiene datos personales? | Sí: el **correo de 4 anotadores** en `Created By` del CSV | No |
| Tratamiento | Se elimina al cargar (`02_verificar_clave.py`) y se sustituye por un hash; **nunca** se guarda en `data/processed` ni se sube a GitHub, Colab o asistentes de IA | — |
| ¿Personas identificables en imágenes? | Manos o dedos en algunas fotos, sin rostros | — |
| ¿Ubicación de fincas privadas? | No. El sitio es un campo experimental universitario publicado en el artículo; el EXIF no trae GPS | Solo el punto institucional del artículo |
| ¿Datos crudos en el repositorio público? | No: `data/raw/` está en `.gitignore`. Solo se publican derivados (descriptores y metadatos sin correos) con atribución CC BY | Sí, derivado público |
| Datos compartidos con IA generativa | Ver `fase1/Bitacora_IA.md` (incluye un incidente declarado) | — |

**Declaración de cumplimiento.** El equipo declara que los datos se tratan conforme a la Ley 1581 de 2012 y a su política de uso de IA.

Firmas: ______________________ · ______________________ · ______________________

## Registro de cambios

| Versión | Fecha | Cambio |
|---|---|---|
| v1 | — | Escenario de referencia RoCoLe con clave `(Calceta, 2019‑04)` |
| **v2** | 2026‑11‑04 | (1) Clave `(Calceta, día de toma)` con EXIF verificado en el 100 % de las fotos. (2) Tabla día × clase y día × teléfono. (3) Revisión de plausibilidad de fechas. (4) Detección de 67 duplicados exactos y partición por grupo pHash. (5) Plan de ablación pre‑registrado para la Fase 2 |

## Referencias

- Kushalappa, A. C., & Eskes, A. B. (Eds.). (1989). *Coffee rust: Epidemiology, resistance, and management*. CRC Press.
- NASA Langley Research Center. (2026). *POWER Data Access* [API]. https://power.larc.nasa.gov
- Parraga‑Alava, J., Cusme, K., Loor, A., & Santander, E. (2019). RoCoLe: A robusta coffee leaf images dataset for evaluation of machine learning based methods in plant diseases recognition. *Data in Brief, 25*, 104414. https://doi.org/10.1016/j.dib.2019.104414
- Parraga‑Alava, J., Cusme, K., Loor, A., & Santander, E. (2019). *RoCoLe: A robusta coffee leaf images dataset* (Version 2) [Data set]. Mendeley Data. https://doi.org/10.17632/c5yvn32dzg.2

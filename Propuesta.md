# Propuesta de Datos · Proyecto Centinela · Fase 1

Una página de texto (secciones 0 a 8) más tres anexos que no cuentan en la extensión: A. prueba de acoplamiento; B. evidencias del test de procedencia; C. checklist de protección de datos (uno por fuente).

## 0. Encabezado

| Campo | Detalle |
| --- | --- |
| Título del proyecto | Centinela del Café: detección de hojas de café robusta afectadas por roya o ácaro rojo |
| Integrantes | Briyith Lisdey Herrera Cruz · Juan Pablo Becerra Tirano · Johnatan Vélez Martínez |
| Fecha | 2026-11-03 |
| Versión | v3 de la Fase 1. La v1 y la v2 usaban imágenes de cacao (Mendeley) sin lugar de captura documentado; por indicación del docente, esta versión cambia la fuente de imagen a RoCoLe, la ruta por defecto del catálogo |

## 1. Problema, cliente y objetivo binario

- **Evento de riesgo y cliente:** a partir de la foto de una hoja, anticipar si una planta de café robusta está afectada, para el técnico de campo de un lote como el del CIIDEA‑ESPAM MFL (Calceta, Manabí, Ecuador), que decide qué plantas inspeccionar.
- **Impacto:** la roya (*Hemileia vastatrix*) defolia la planta y reduce la cosecha siguiente, y si se detecta tarde se dispersa a las plantas vecinas; el ácaro rojo broncea la hoja y reduce la fotosíntesis.
- **Objetivo binario:** «hoja afectada: sí/no».
  - RoCoLe trae 6 clases ordinales (sana, ácaro rojo, roya 1 a 4). Se agrupan en **sana = 0** frente a **ácaro o roya de cualquier nivel = 1**.
  - Justificación por costo: para el técnico, toda hoja afectada exige inspección, y la severidad no cambia la acción inmediata.
  - **Se pierde la severidad.** Se conserva como variable para el análisis de errores por subgrupo.
  - Balance: 769 afectadas frente a 791 sanas (49,3 %).

## 2. Las dos modalidades del sistema completo

- **Rama de imagen:** cada imagen es la foto de una hoja (haz o envés) tomada con smartphone en campo. Aporta la forma, el color y la textura de las pústulas de roya y del bronceado del ácaro.
- **Rama temporal:** temperatura (T2M, T2M_MAX, T2M_MIN), humedad relativa (RH2M), precipitación (PRECTOTCORR), viento (WS2M) y radiación (ALLSKY_SFC_SW_DWN). Vienen del reanálisis satelital NASA POWER, con frecuencia diaria.
- **Pregunta de control, «¿La imagen contiene algo que un número no contenga?»:** sí. La lesión visible en una hoja concreta no está en ningún registro de clima. Son dos instrumentos independientes: una cámara y un modelo meteorológico satelital.

## 3. Fuentes (con enlaces y licencia)

| Modalidad | Fuente (productor primario) | Enlace o DOI | Licencia y versión | Dónde se verificó la licencia | ¿Datos de personas? |
| --- | --- | --- | --- | --- | --- |
| Imagen | RoCoLe v2 (Parraga‑Alava et al., ESPAM MFL; Mendeley Data) | https://doi.org/10.17632/c5yvn32dzg.2 | CC BY 4.0 | Ficha de la API de Mendeley Data (`data_licence: CC BY 4.0`), consultada el 2026‑11‑03 | Las fotos son hojas de café, sin personas. El archivo de anotaciones trae el correo de 4 anotadores: la columna se elimina al cargar (anexo C) |
| Temporal | NASA POWER (NASA Langley Research Center) | https://power.larc.nasa.gov/ | Datos NASA de libre uso, sin token | Página de la fuente | No |

## 4. Por qué NO es un reto resuelto

- **El dato es crudo:**
  - 22,2 % de celdas vacías en el CSV de anotaciones (4 de 18 columnas).
  - Tres resoluciones de imagen y fondos de campo.
  - Un nombre de archivo fuera de convención (`C11P11HE.jpg`).
  - **67 fotos repetidas byte a byte con nombres de árboles distintos**: en 27 la severidad difiere entre copias y en 1 se contradice la etiqueta binaria.
- **Procedencia:** se descarga de Mendeley (fuente primaria, publicada el 2019‑05‑17), con la huella SHA‑256 de cada archivo contrastada con la que publica su API. **Existen 3 réplicas en Kaggle** (búsqueda del 2026‑11‑03, anexo B): no se usan, y su contraste está documentado.
- **No hay una solución que baste copiar:** el escenario adoptado es el de referencia del curso, pero lo que se entrega no existe publicado:
  - la clave por día de toma verificada sobre las 1 560 fotos;
  - la deduplicación y la partición por grupo pHash;
  - el umbral por costo y la auditoría reproducida.

## 5. Plan de la Fase 1, clave de alineación y prueba de acoplamiento

- **Modalidad que se modela en la Fase 1 (aplanada a tabla):** la imagen, resumida en 24 descriptores calculados **solo dentro del polígono de la hoja** anotado en RoCoLe: media y desviación RGB y HSV, ExG, VARI, `frac_roya` (fracción de píxeles con tono de pústula), fracciones amarilla, oscura y verde, percentiles de brillo, gradiente y área. Entrada de un MLP de 2 capas ocultas.
- **Paso a la Fase 2:** una CNN reemplaza a los descriptores; una LSTM/GRU procesa la serie diaria de NASA POWER (60 días × 7 variables) y ambas ramas se fusionan, con ablación imagen / clima / fusión.
- **Clave de alineación (se marca una):**
  - [x] Agregada, p. ej., (región, mes), con su salvedad de fuga de datos y de rasgo casi constante: **(sitio, día de toma) = (CIIDEA‑Calceta, `DateTimeOriginal` del EXIF)**, con la ventana de **60 días que termina el día anterior** a la toma. Esa ventana cubre el periodo latente de la roya, de 3 a 5 semanas.
    - La fecha está en el EXIF del 100 % de las fotos (1 560/1 560) y el GPS en ninguna (0/1 560). El sitio es el que documenta el artículo de datos.
    - Hay **5 días de toma** (2019‑04‑09 a 2019‑05‑02).
    - *Salvedad de fuga:* la clase **no** depende del día (47–52 % de afectadas cada día; V de Cramér = 0,024), pero el teléfono sí cambia con el día (V = 0,69). Por eso se hacen pruebas dejando un día completo fuera.
    - *Salvedad de rasgo casi constante:* hay solo 5 vectores de clima casi iguales, así que en la Fase 2 se espera que el clima no ayude a clasificar la hoja. Si se confirma, se reportará como hallazgo.
  - [ ] Por muestra, p. ej., (finca, semana).
- **Prueba de acoplamiento (obligatoria):** diez filas reales en el anexo A.
- **Partición (modalidad imagen):** grupos = árboles unidos por pHash (336 grupos); `StratifiedGroupKFold` con 20 pliegues → **69,3 / 15,4 / 15,3 %**; verificación de que no hay árboles, SHA‑256 ni pHash ≤ 6 bits compartidos entre conjuntos.

## 6. Línea base comprometida

- **Clase mayoritaria:** exactitud y *recall* de la clase de riesgo en el conjunto de prueba (el *recall* vale 0 cuando la clase mayoritaria es «sin riesgo»).
- **Escalar crudo** (solo si la modalidad aplanada resume un escalar): **`frac_roya`**, la fracción de píxeles de la hoja con tono de pústula, usada sola, con su umbral elegido en validación (por F1 y por la misma regla de costo del MLP).
- La partición (semilla 42, 20 pliegues por grupo), la semilla y estas cifras se conservan: son el punto de comparación de la Fase 2.

## 7. Riesgos éticos y protección de datos

| Riesgo | Daño para el cliente | Mitigación |
| --- | --- | --- |
| Sesgo de muestreo o representatividad | Un solo lote de robusta en la costa ecuatoriana, 4 teléfonos Samsung y 5 días de 2019. El modelo puede fallar en arábica, en otras altitudes o con otras cámaras | Se declara el alcance; se reporta el desempeño por teléfono y por día y se prueba dejando un día fuera |
| Falsa alarma | Inspecciones innecesarias: tiempo del técnico | Umbral por costo elegido en validación; tabla de sensibilidad del umbral para el cliente |
| Alarma omitida | Una hoja con roya declarada sana: el foco crece y se dispersa | Costo FN = 5 × FP; sensibilidad reportada por severidad (roya 1 incipiente es el subgrupo crítico) |

El checklist de protección de datos va en el anexo C, uno por fuente.

## 8. Cita APA 7 de cada fuente

- Parraga‑Alava, J., Cusme, K., Loor, A. y Santander, E. (2019). *RoCoLe: A robusta coffee leaf images dataset* (Versión 2) [Conjunto de datos]. Mendeley Data. https://doi.org/10.17632/c5yvn32dzg.2
- Parraga‑Alava, J., Cusme, K., Loor, A. y Santander, E. (2019). RoCoLe: A robusta coffee leaf images dataset for evaluation of machine learning based methods in plant diseases recognition. *Data in Brief, 25*, 104414. https://doi.org/10.1016/j.dib.2019.104414
- National Aeronautics and Space Administration. (s. f.). *POWER: Prediction Of Worldwide Energy Resources* [Conjunto de datos]. https://power.larc.nasa.gov/

---

## Anexo A · Prueba de acoplamiento

Clave: (CIIDEA‑Calceta [−0,834206, −80,177159], día de toma del EXIF); ventana de 60 días que termina el día anterior.

| N.º | Clave | Dato de la rama de imagen (archivo o rasgo) | Dato de la serie (variable y valor) |
| --- | --- | --- | --- |
| 1 | (Calceta, 2019‑04‑09) | `C9P11H2.jpg` · sana · EXIF 09:48:05 · SM‑J200M | PRECTOTCORR acumulada 2019‑02‑08…04‑08 = 643,8 mm · T2M media 25,80 °C · RH2M 85,8 % |
| 2 | (Calceta, 2019‑04‑09) | `C9P7H1.jpg` · roya 1 · 09:40:31 · SM‑J200M | 643,8 mm · 25,80 °C · 85,8 % |
| 3 | (Calceta, 2019‑04‑10) | `C6P32H1.jpg` · roya 1 · 09:37:03 · SM‑J200M | 2019‑02‑09…04‑09: 641,9 mm · 25,79 °C · 85,9 % |
| 4 | (Calceta, 2019‑04‑10) | `C7P20E2.jpg` · roya 2 · 10:23:08 · SM‑J200M | 641,9 mm · 25,79 °C · 85,9 % |
| 5 | (Calceta, 2019‑04‑11) | `C3P37E2.jpg` · ácaro rojo · 10:52:01 · SM‑G610M | 2019‑02‑10…04‑10: 627,3 mm · 25,79 °C · 85,8 % |
| 6 | (Calceta, 2019‑04‑11) | `C10P38E2.jpg` · roya 1 · 10:05:16 · SM‑J700M | 627,3 mm · 25,79 °C · 85,8 % |
| 7 | (Calceta, 2019‑05‑01) | `C5P18H1.jpg` · sana · 11:53:33 · SM‑J200M | 2019‑03‑02…04‑30: 518,4 mm · 25,67 °C · 86,2 % |
| 8 | (Calceta, 2019‑05‑01) | `C5P26E2.jpg` · roya 2 · 12:05:01 · SM‑J200M | 518,4 mm · 25,67 °C · 86,2 % |
| 9 | (Calceta, 2019‑05‑02) | `C1P25E2.jpg` · roya 1 · 10:43:20 · SM‑J320H | 2019‑03‑03…05‑01: 517,8 mm · 25,66 °C · 86,2 % |
| 10 | (Calceta, 2019‑05‑02) | `C1P10E1.jpg` · sana · 10:16:11 · SM‑J320H | 517,8 mm · 25,66 °C · 86,2 % |

Código o consulta que produjo las filas: `scripts/02_verificar_clave.py`, secciones 2 (EXIF) y 6 (API `https://power.larc.nasa.gov/api/temporal/daily/point?parameters=T2M,T2M_MAX,T2M_MIN,RH2M,PRECTOTCORR,WS2M,ALLSKY_SFC_SW_DWN&community=AG&latitude=-0.834206&longitude=-80.177159&start=20190207&end=20190502&format=JSON`). Salida: `resultados/anexo_A_10_filas.csv`. Fotos por día y clase: `resultados/tabla_dia_x_clase_*.csv`.

## Anexo B · Evidencias del test de procedencia

1. **Cadena de procedencia** (un archivo por fila). Las 1 560 fotos, con su huella, están en `data/manifest_mendeley.csv`.

| Archivo (nombre exacto) | URL o DOI del productor primario | Fecha de descarga | Huella SHA-256 |
| --- | --- | --- | --- |
| `c5yvn32dzg-2.zip` (ZIP oficial de la v2, nombre en el servidor; guardado como `data/rocole_v2_mendeley.zip`; 2 245 588 288 bytes) | https://data.mendeley.com/public-api/zip/c5yvn32dzg/download/2 (DOI 10.17632/c5yvn32dzg.2) | 2026-11-03 | `31daa765fb0cc27d4b9b897fa6350aacf0087ee5771d3c7599d2ac7041b866ff` |
| `RoCoLE-csv.csv` | ídem | 2026-11-03 | `a9aa6b411569dba7e9ba56fed79e5cd82af04a0ef092e56309158d632ac5205b` (= huella publicada por Mendeley) |
| `RoCoLe-classes.xlsx` | ídem | 2026-11-03 | `0040a57ffd441bd69ead997309d14c8d7f6fdfc83c5dbdc47805e0bc4da05ef2` (= Mendeley) |
| `RoCoLE-json.json` | ídem | 2026-11-03 | `286b73725b7c027a6bc9a1118a281b026e1ec9e35c3510398ed6bbc0e579cf99` (= Mendeley) |
| `RoCoLE-coco.json` | ídem | 2026-11-03 | `beac06144c2865cd0ceb2ef3f81791334cf1674400184ff1aa123d1e67f7ec4f` (= Mendeley) |
| 1 560 fotos `C#P#[H\|E]#.jpg` | ídem | 2026-11-03 | En `data/manifest_mendeley.csv`. 1 004 contrastadas con la huella publicada, **0 diferencias**; las 556 restantes solo tienen huella calculada, porque la API lista como máximo 1 000 archivos por carpeta |
| `nasa_power_calceta_diario.csv` | API NASA POWER (URL del anexo A) | 2026-11-03 | Se recalcula al ejecutar `02_verificar_clave.py` |

2. **Descarga trazable:**
   - `scripts/01_descarga_verificacion.py` descarga el ZIP de la URL de Mendeley y calcula `hashlib.sha256` de cada archivo.
   - El cuaderno lee `data/processed/descriptores_rocole.csv`, derivado de ese ZIP y publicado en el repositorio con su procedencia.
   - Resultado de `grep -inE "kaggle|kagglehub|/kaggle/input" *.ipynb` en el cuaderno y los scripts del pipeline: **0 coincidencias** (2026-11-03). El único archivo que menciona Kaggle es `scripts/04_contraste_espejos_kaggle.py`, que solo lee metadatos públicos para este anexo y no descarga datos.

3. **Búsqueda declarada de espejos** (2026-11-03; `site:kaggle.com "RoCoLe"` y `site:kaggle.com "robusta coffee leaf"`):
   - https://www.kaggle.com/datasets/diegopgonzlez/rocoleoriginal
   - https://www.kaggle.com/datasets/nirmalsankalana/rocole-a-robusta-coffee-leaf-images-dataset
   - https://www.kaggle.com/datasets/jumenta/rocole-test

4. **Huella de crudeza** (antes de limpiar), sobre el CSV de anotaciones `RoCoLE-csv.csv`:
   - Tamaño: 1 560 filas × 18 columnas.
   - Faltantes: **22,2 %** de celdas vacías (`Agreement` 1 545/1 560; `Benchmark Agreement`, `Benchmark ID` y `Benchmark Reference ID` 1 560/1 560).
   - Balance de la etiqueta: sana 791 / afectada 769 (49,3 %). En 6 clases: 791 / 167 / 344 / 166 / 62 / 30.
   - Además: 3 resoluciones, 67 fotos duplicadas byte a byte con nombres de árboles distintos, 1 nombre fuera de convención y correos de anotadores.

5. **Contraste con el espejo**, con los metadatos públicos de la API de Kaggle, sin descargar archivos (`scripts/04_contraste_espejos_kaggle.py` → `resultados/contraste_espejos_kaggle.csv`):

| Espejo | Licencia que declara | Contenido | Contraste con la fuente primaria |
| --- | --- | --- | --- |
| diegopgonzlez/rocoleoriginal (subido el 2021‑04‑24) | CC BY‑SA 4.0 (**distinta** de la original, CC BY 4.0) | 4 684 archivos; las 1 560 fotos tienen el mismo nombre y tamaño que en Mendeley | Es una **resubida exacta** de la fuente primaria, posterior a ella (2019‑05‑17), con la licencia mal declarada. No se puede mostrar una huella distinta: la procedencia se demuestra por la cadena (descarga desde el endpoint de Mendeley y SHA‑256 contrastado con su API), no por diferencia de contenido. **No se usa** |
| nirmalsankalana/rocole‑a‑robusta‑coffee‑leaf‑images‑dataset (2025‑01‑25) | MIT (incorrecta) | 1 560 fotos **recomprimidas** (0 de 1 560 con el mismo tamaño; 407 MB frente a 1,57 GB), en carpetas por clase y **sin anotaciones** | Huella distinta y estructura distinta; no trae el polígono de la hoja que usa este proyecto |
| jumenta/rocole‑test (2020‑08‑24) | CC BY 4.0 | Solo 315 fotos («test»), en carpetas anidadas | Subconjunto parcial; no es la fuente usada |

## Anexo C · Checklist de protección de datos personales (uno por fuente)

### C.1 · Fuente: RoCoLe v2 (Mendeley Data)

Regla de la Fase 1: sin personas identificables. Solo se usan datos públicos, agregados o anonimizados por la fuente oficial. «Ley» = Ley 1581 de 2012; «DUR» = Decreto 1074 de 2015, que compila el Decreto 1377 de 2013.

**Resultado** (se marca uno):
- [x] Sin datos de personas. *En los datos que usa el proyecto. La fuente incluye, en `RoCoLE-csv.csv`, la columna `Created By` con el correo de 4 anotadores; se elimina al cargar y no aparece en ningún derivado.*
- [ ] Datos de personas anonimizados por la fuente.
- [ ] Datos personales o sensibles → no admisibles en la Fase 1; en fases posteriores requieren el visto bueno del docente antes de modelar.

#### A. ¿Hay datos personales? (Ley, art. 3 lit. c)
- [x] Se revisaron todas las columnas, imágenes y metadatos en busca de identificadores directos. Hallazgo: los **correos** en `Created By` (identificador directo); se eliminan. Algunas fotos muestran dedos o manos, sin rostro.
- [x] Se revisaron los identificadores indirectos: el **EXIF no trae GPS** (0/1 560). La fecha y el modelo de teléfono no señalan a ninguna persona; el sitio es un campo experimental universitario, no una finca privada.
- [x] Un identificador seudonimizado se trata como dato personal. El hash del anotador (`anot_xxxxxx`) se conserva solo en `metadatos_rocole.csv` para auditar la anotación; no entra al modelo.

#### B. ¿Hay datos sensibles? (Ley, art. 5)
- [x] No hay datos sensibles (no hay salud, biometría ni rostros).
- [x] No aplica: no son imágenes médicas ni conteos de salud.

#### C. ¿Hay niños, niñas o adolescentes? (Ley, art. 7; DUR 2.2.2.25.2.9)
- [x] No hay titulares menores; no se recolectan datos propios.

#### D. Base legal o autorización para datos de terceros
- [x] Fuente pública con licencia CC BY 4.0, citada (§3 y §8).
- [x] No aplica: no hay datos propios ni de una organización.
- [x] No aplica: no hay datos de un empleador.

#### E. Finalidad (Ley, art. 4 lit. b)
- [x] Finalidad declarada: «uso académico para entrenar y evaluar un modelo en el curso; no se toman decisiones sobre personas identificadas».

#### F. Anonimización y riesgo de reidentificación
- [x] Se eliminaron los identificadores directos: los correos de `Created By`.
- [x] Las imágenes no traen GPS. Las fotos crudas no se suben a Colab ni a GitHub (`data/raw/` va en `.gitignore`); el cuaderno usa descriptores tabulares. La fecha del EXIF se conserva porque es la clave de alineación y no identifica a nadie.
- [x] Prueba de reidentificación: «¿podría alguien identificar a una persona combinando estas columnas con información pública?» → No. Los derivados contienen archivo, árbol, clase, día, modelo de teléfono y descriptores de color, sin personas.

#### G. Licencia de uso (de cada fuente, verificada en la fuente primaria)
- [x] CC BY 4.0, verificada en la ficha de la API de Mendeley Data del DOI 10.17632/c5yvn32dzg.2 (2026-11-03).
- [x] Obligaciones BY: atribuir e indicar los cambios. Cambios declarados en el README: «características tabulares derivadas de RoCoLe; columna de correos eliminada; 68 fotos excluidas por duplicación».
- [x] Se cumplen los términos del portal (cita obligatoria; no se sugiere aval de los autores).

#### H. Transferencia o transmisión internacional (Colab, Drive, GitHub, asistentes de IA)
- [x] Se sabe que estos servicios procesan los datos fuera de Colombia (Ley, art. 26).
- [x] A Colab y a GitHub solo se suben derivados sin personas identificables.
- [ ] A los asistentes de IA no se les pegan registros reales con datos personales. **No se cumplió una vez:** durante la exploración, el asistente de IA imprimió una fila completa del CSV, que incluía un correo de anotador. Está declarado en la Bitácora (E-4); desde entonces la columna se borra antes de mostrar cualquier fila.

#### I. Minimización y conservación
- [x] Solo se conservan las columnas necesarias. Del CSV se usan `External ID` y el polígono `Label`; las 4 columnas vacías y el correo se descartan.
- [x] Los datos crudos se guardan en `data/raw/` en el equipo local y se borran al terminar la Fase 3.

#### J. Publicación del repositorio (Ley, art. 4 lit. f)
- [x] El repositorio no contiene datos personales y la licencia CC BY 4.0 permite redistribuir los derivados con atribución.
- [x] El repositorio trae el script de descarga desde Mendeley; `data/raw/` va en `.gitignore`.
- [x] Las salidas del cuaderno no muestran correos: se verificó con una búsqueda de `@…` en el repositorio, sin coincidencias.

### C.2 · Fuente: NASA POWER

**Resultado:**
- [x] Sin datos de personas.
- [ ] Datos de personas anonimizados por la fuente.
- [ ] Datos personales o sensibles.

- **A.** [x] Se revisaron todas las columnas (variables meteorológicas por fecha): no hay identificadores. [x] La única coordenada es la del campo experimental publicada en el artículo de datos.
- **B.** [x] No hay datos sensibles.
- **C.** [x] No hay menores.
- **D.** [x] Fuente pública oficial (datos NASA de libre uso), citada.
- **E.** [x] Finalidad académica.
- **F.** [x] No aplica: no hay identificadores.
- **G.** [x] Libre uso con atribución, verificado en https://power.larc.nasa.gov/. [x] Se cita la fuente.
- **H.** [x] Se pueden subir a Colab y GitHub (dato público).
- **I.** [x] Solo 7 variables diarias para el periodo necesario.
- **J.** [x] Se redistribuye el CSV diario con atribución.

**Declaración.** Lo consignado en este checklist es verdadero y se revisó para cada fuente de la Propuesta.

| Integrante | Firma | Fecha (AAAA-MM-DD) |
| --- | --- | --- |
| Briyith Lisdey | | |
| Juan Pablo | | |
| Johnatan | | |

# cafe-centinela

# Propuesta de Datos — Proyecto Centinela (Fase 1)
**Curso:** Profundización I: Redes Neuronales — Deep Learning (EA-N-F-004) · USTA  
**Proyecto:** Centinela del Café — Sistema de Alerta Temprana Fitosanitaria en Hoja de Café  
**Estado de Compuerta:** Versión para Aprobación · Fecha: Noviembre 2026  

---

### 1. Problema y Cliente No Especializado
* **Cliente:** Comité Técnico y Cooperativa de Pequeños Productores Cafeteros (extensible de productores de *Coffea canephora* en Manabí, Ecuador, a cooperativas cafeteras en Antioquia y el Eje Cafetero, Colombia).
* **Problema de negocio:** La roya del cafeto (*Hemileia vastatrix*) y la araña roja (*Tetranychus urticae*) provocan defoliación severa y pérdidas de rendimiento de entre el 30 % y el 80 % si no se detectan en fases tempranas. Las inspecciones visuales manuales en lotes campesinos son subjetivas, lentas y costosas.
* **Objetivo Binario ($Y \in \{0, 1\}$):** Predecir el **evento de riesgo fitosanitario** a nivel foliar:
  * $Y = 0$ (**Hoja Sana**): Categoría original `healthy` (sin clorosis, pústulas ni ácaros visibles; $n = 791$ imágenes).
  * $Y = 1$ (**Hoja en Riesgo / Enferma**): Agrupación justificada de los estados patológicos del dataset (`rust_level_1`, `rust_level_2`, `rust_level_3`, `rust_level_4` y `red_spider_mite`; $n = 769$ imágenes). En un sistema Centinela de alerta temprana, cualquier grado de infección o plaga exige activar protocolo de vigilancia fitosanitaria en el lote.

### 2. Dos Modalidades Reales e Instrumentos Independientes

| Modalidad | Rol por Fase | Instrumento de Captura | Fuente Primaria Oficial (No-Kaggle) | Licencia |
| :--- | :--- | :--- | :--- | :--- |
| **Modalidad 1: Imagen RGB Foliar** | **Fase 1 (Aplanada a Tabular)** y **Fase 2 (CNN sobre píxeles)** | Cámara óptica de teléfono inteligente a 20–30 cm en condiciones reales de campo (haz y envés de 390 plantas, 1.560 fotos). | **Mendeley Data (Fuente Primaria):** Párraga-Alava et al. (2019). *RoCoLe: A robusta coffee leaf images dataset*. DOI: `10.17632/c5yvn32dzg.2` (`https://data.mendeley.com/datasets/c5yvn32dzg/2`). | **CC BY 4.0** |
| **Modalidad 2: Serie Agrometeorológica** | **Fase 2 (RNN / LSTM / GRU + Fusión Multimodal)** | Sensores satelitales y reanálisis meteorológico MERRA-2 (temperatura a 2m, humedad relativa, precipitación y radiación solar diaria). | **NASA POWER Agroclimatology Archive:** `https://power.larc.nasa.gov/` (Coordenadas del cultivo experimental: Calceta, Manabí, Lat `-0.845°`, Lon `-80.164°`). | **Dominio Público Abierto (NASA)** |

* **Verificación de independencia modal:** La fotografía RGB contiene patrones espaciales de pústulas uredosporas, necrosis y textura foliar que ningún sensor meteorológico puede medir; a su vez, la serie climática captura la ventana de incubación fúngica (humedad mayor al 85 % y temperaturas entre 21 °C y 25 °C) que una foto estática no registra.

### 3. Aplanamiento Tabular en Fase 1 y Clave de Alineación Honesta
* **Versión tabular en Fase 1:** De cada imagen RGB de RoCoLe se extrae un vector numérico $\mathbf{x} \in \mathbb{R}^{14}$ compuesto por promedios y desviaciones estándar por canal ($R, G, B$), estadísticos en espacio $HSV$, proporción de área foliar con tonalidad clorótica/necrótica (`rust_pixel_ratio`), índices espectrales agronómicos (**ExG**, **ExR**, **NGRDI**) y descriptores de textura espacial (varianza del Laplaciano y entropía).
* **Clave de Alineación (Imagen ↔ Clima para Fase 2):** Alineación **agregada espacio-temporal** por región geográfica (`Lat -0.845, Lon -80.164`, Campus Politécnico ESPAM MFL, Ecuador) y ventana temporal del campaña de muestreo (2018–2019). **Declaración de datos honestos:** Una fotografía individual de hoja no posee una estación meteorológica exclusiva; por tanto, la fusión en Fase 2 asociará el estado foliar con el régimen climático agregado de los 30 días previos al muestreo.

### 4. ¿Por qué no es un reto resuelto y por qué exige EDA?
Las fotografías de RoCoLe fueron tomadas directamente en campo sin cabina de iluminación controlada: presentan fondos heterogéneos (suelo, maleza, manos del agricultor, otras hojas), reflejos especulares del sol, sombras duras y casos incipientes de roya (`rust_level_1` con menos del 5 % del área foliar afectada) donde el promedio global de color se confunde con hojas sanas. El EDA es indispensable para auditar sesgos de iluminación, colinealidad entre canales ópticos y el efecto de dilución espacial al promediar píxeles.

### 5. Riesgos Éticos y Declaración de Protección de Datos Personales (Ley 1581 de 2012)
* **Cumplimiento Ley 1581 de 2012 (Colombia):** El conjunto RoCoLe y las series NASA POWER contienen exclusivamente mediciones biológicas vegetales y variables geofísicas; **no recolectan ni procesan datos personales sensibles ni identificables (PII)**. En un futuro despliegue comercial en fincas colombianas, la geolocalización de predios y cédulas cafeteras requerirá autorización expresa del titular conforme a la Ley 1581 de 2012 y el Decreto 1377 de 2013.
* **Riesgos éticos en el agro:** Un **Falso Negativo** (no alertar roya presente) expone al pequeño caficultor a la pérdida de su cosecha y contagio a fincas vecinas; un **Falso Positivo** induce la aplicación innecesaria de fungicidas cúpricos o triazoles, elevando costos de producción y contaminando suelos y fuentes hídricas. Además, existe riesgo de **sesgo de dominio (*domain shift*)** si el modelo entrenado en *Coffea canephora* (Robusta, zona baja tropical) se aplica sin recalibración sobre *Coffea arabica* (variedades Castillo o Caturra en ladera andina colombiana).

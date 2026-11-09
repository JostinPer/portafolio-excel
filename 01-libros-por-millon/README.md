# 📚 Libros nuevos por millón de habitantes: caso de depuración de datos

- **Herramientas:** Microsoft Excel · Power Query · Tablas dinámicas · Segmentadores · Fórmulas de control
- **Autor:** Antony Pérez Parrales
- **Tipo de proyecto:** práctica con datos públicos

---

## Resumen

Al trabajar con un dataset público sobre la publicación de libros por país, descubrí que **el 74% de los valores estaban corruptos**: los decimales se habían perdido y los números quedaron multiplicados o divididos por 10, 100 o más. Identifiqué la causa, reemplacé los datos por la fuente original y construí un proceso automatizado en Power Query con una hoja de controles que verifica la calidad de los datos cada vez que se actualiza.

| Indicador | Antes | Después |
|---|---|---|
| Registros correctos | 2,134 de 8,312 (26%) | 8,312 de 8,312 (100%) |
| Actualización de datos | Copiar y pegar manualmente | Un clic en *Actualizar todo* |
| Verificación de calidad | Ninguna | 8 controles automáticos |

### Contenido

1. [El problema](#1-el-problema)
2. [El proceso](#2-el-proceso)
3. [Resultado](#3-resultado)
4. [Errores encontrados en el camino](#4-errores-encontrados-en-el-camino-y-cómo-los-resolví)
5. [Aprendizajes](#5-aprendizajes)
6. [Estructura del proyecto](#estructura-del-proyecto)
7. [Cómo usar el archivo](#cómo-usar-el-archivo)

---

## 1. El problema

Descargué de Kaggle una versión del dataset *New book titles per million people*. Al cargarlo y graficarlo, las series mostraban saltos imposibles. Por ejemplo, Austria:

| Año | Valor en el CSV de Kaggle | Valor real |
|---|---|---|
| 1958 | 4.416.786.191.219.800 | 441.68 |
| 1959 | 387.827.135.102.924 | 387.83 |
| 1960 | 4.637.644.103.766.170 | 463.76 |

![CSV corrupto descargado de Kaggle](capturas/01-csv-corrupto.png)

*El CSV de Kaggle abierto en Excel: los puntos aparecen cada tres cifras y el decimal real se perdió. Solo los valores con un único punto, como el 25.6 de Bélgica, fueron reconocidos como números.*

**Causa:** el archivo había sido guardado con una configuración regional que usa el punto como separador de miles. Los puntos se insertaron cada tres cifras y la posición real del decimal se perdió.

**Conclusión clave:** estos datos no se podían reparar desde el propio archivo, porque la información del decimal ya no existía. La solución correcta era volver a la fuente.

---

## 2. El proceso

### Datos

- **Fuente original:** [Our World in Data](https://ourworldindata.org/grapher/new-books-per-million.csv)
- **Registros:** 8,312
- **Cobertura:** 71 países y entidades, entre los años 1500 y 2009
- **Unidad:** títulos nuevos publicados por cada millón de habitantes

### Power Query

La consulta principal, `Nuevos libros por millones`, lee el CSV directamente desde la carpeta del proyecto y aplica estos pasos:

1. **Origen:** lectura del CSV con codificación UTF-8 (65001).
2. **Encabezados promovidos.**
3. **Cambio de tipo con configuración regional Inglés (EE. UU.):** garantiza que el punto se interprete como decimal en cualquier computadora, sin importar su configuración regional.
4. **Columnas renombradas al español:** País, Codigo, Año, Libros nuevos por millones de personas.
5. **Año convertido a número entero.**

Una segunda consulta, `Filas_origen`, cuenta las filas del CSV original para compararlas con el resultado final.

![Pasos de Power Query](capturas/02-power-query.png)

*Consulta en Power Query: lectura del CSV con codificación UTF-8 y pasos aplicados.*

### Hoja de controles

Verificaciones automáticas que se recalculan al actualizar. La tabla de datos limpios se llama `Nuevos_libros_por_millones`:

| Control | Qué verifica | Fórmula |
|---|---|---|
| Filas en la tabla limpia | Que coincidan con las filas del CSV original | `=FILAS(Nuevos_libros_por_millones[])` comparada con `Filas_origen` |
| Celdas vacías | Que no haya datos faltantes | `=CONTAR.BLANCO(Nuevos_libros_por_millones[])` |
| Valores no numéricos | Que todos los valores sean números válidos | `=FILAS(...) - CONTAR([Libros nuevos por millones de personas])` |
| Duplicados país-año | Que cada país aparezca una sola vez por año | `=FILAS(...) - FILAS(UNICOS(Nuevos_libros_por_millones[[País]:[Año]]))` |
| Valor mínimo | Que no haya valores negativos | `=MIN(...)` ≥ 0 |
| Valor máximo | Que no haya valores absurdos | `=MAX(...)` < 10,000 |
| Primer año | Que no haya años anteriores a 1500 | `=MIN(Nuevos_libros_por_millones[Año])` |
| Último año | Que los años estén entre 1500 y 2026 | `=MAX(Nuevos_libros_por_millones[Año])` |

Una celda de **estado general** resume todos los controles en un solo mensaje.

![Controles en verde](capturas/03-controles.png)

*Hoja de controles con los datos correctos: todos los controles en verde.*

**Prueba de los controles:** alteré a propósito una copia del CSV (una fila duplicada, celdas vacías, un valor no numérico y un valor negativo), apunté la consulta a esa copia y confirmé que cada control detectaba su error correspondiente.

> El archivo `Datos/new-books-per-million_PRUEBA-controles.csv` es la copia alterada usada en esta prueba. **No contiene datos reales**: se conserva solo como evidencia de que los controles funcionan.

![Controles detectando errores](capturas/04-controles-prueba.png)

*Prueba con datos alterados: los controles detectan la fila extra, las celdas vacías, el valor no numérico, el duplicado y el valor negativo.*

### Visualización

- Tabla dinámica con el **promedio** por país y año.
- Gráfico de líneas con la tendencia por año.
- Segmentador para filtrar por país.

---

## 3. Resultado

- **8,312 registros verificados**, idénticos a la fuente original.
- Proceso **automatizado**: al reemplazar el CSV por una versión nueva, basta con *Datos > Actualizar todo*.
- **Todos los controles en verde.**

![Visualización final](capturas/05-visualizacion.png)

*Tabla dinámica, gráfico y segmentador por país. En Armenia se observa una caída marcada entre 1942 y 1945, durante la Segunda Guerra Mundial.*

---

## 4. Errores encontrados en el camino y cómo los resolví

Documento también los errores que cometí durante el proyecto, porque fueron parte del aprendizaje:

| Error | Causa | Solución |
|---|---|---|
| Valores con magnitudes incorrectas | Intenté reparar los números tomando el texto antes del segundo punto, sin saber que la posición del decimal se había perdido | Reemplazar los datos por la fuente original |
| La tabla dinámica sumaba una tasa | Sumar "libros por millón" de varios años no tiene significado | Cambiar el resumen a **Promedio** |
| El control "Primer año" daba 0 | Al quitar un paso automático de Power Query, la columna Año quedó como texto y `MIN` la ignoraba | Convertir Año a número entero y mejorar el control de "Último año" para revisar ambos límites |
| La tabla dinámica perdió su columna de valores | Cambié el nombre de la columna en Power Query y la tabla dinámica dejó de encontrarla | Volver a agregar el campo y revisar la presentación después de cada cambio en la consulta |

---

## 5. Aprendizajes

- **Nunca confiar en un dataset sin verificarlo contra su origen.** Un archivo puede verse bien y estar dañado.
- **Abrir un CSV en un editor de texto antes que en Excel** permite ver los separadores reales.
- **Las tasas no se suman**: el tipo de resumen depende de lo que mide el dato.
- **Un control que siempre da verde no sirve**: hay que probarlo con datos alterados a propósito.
- **Cualquier cambio en el origen puede romper lo que depende de él**, como tablas dinámicas y gráficos.

---

## Estructura del proyecto

```
01-libros-por-millon/
├── 01-Libros-por-millon.xlsx                     ← libro de trabajo
├── README.md                                     ← este documento
├── Datos/
│   ├── new-books-per-million.csv                 ← datos originales de Our World in Data
│   └── new-books-per-million_PRUEBA-controles.csv ← copia alterada para probar los controles
└── capturas/                                     ← imágenes de este README
```

**Hojas del libro:**

| Hoja | Contenido |
|---|---|
| Visualizacion de datos | Tabla dinámica, gráfico de líneas y segmentador por país |
| Controles | Verificaciones automáticas de calidad y notas del dataset |
| new-books-per-million | Datos limpios cargados desde Power Query |

El CSV corrupto de Kaggle no se incluye: se documenta con la captura de la sección 1.

---

## Cómo usar el archivo

**Requisitos:** Microsoft Excel 365 o Excel 2021 en adelante (la función `UNICOS` de los controles no existe en versiones anteriores).

1. Descarga la carpeta completa del proyecto (el Excel y la carpeta `Datos`).
2. Abre el Excel y ve a *Datos > Obtener datos > Configuración de origen de datos*.
3. Selecciona la ruta del CSV, haz clic en **Cambiar origen** y apunta al archivo `Datos/new-books-per-million.csv` en tu computadora. Como las dos consultas usan la misma ruta, ambas se actualizan con este cambio.
   - **Alternativa:** también se puede cambiar la ruta desde el Editor de Power Query, con el engranaje del paso *Origen* de cada consulta. En ese caso hay que hacerlo en las dos consultas (`Nuevos libros por millones` y `Filas_origen`).
4. Presiona *Datos > Actualizar todo*.
5. Revisa la hoja **Controles**: debe decir *Todos los controles OK*.

---

## Mejoras pendientes

- Agregar una hoja *Inicio* con el objetivo del libro, el contenido de cada hoja y las instrucciones de uso.
- Leer la ruta del CSV desde una celda como parámetro, para que no haya que cambiarla manualmente.
- Agregar un control que detecte saltos bruscos entre años consecutivos de un mismo país.

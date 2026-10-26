# Informe — Clasificación de variables y población vs. muestra

**Para:** Gerencia de RetailAndina S.A. · **De:** Equipo de Business Intelligence (Joel Champutiz)
**Fecha:** 23/10/2026 · **Asunto:** Base de fidelización: tipos de variables y estimación por muestreo

---

## Resumen ejecutivo

Se analizaron los datos del sistema de fidelización compuesto por 400 clientes mediante la clasificación de las variables presentes en Python con `pandas`. Además, se realizó un análisis de tres diseños de muestreo con $n=20$ (conveniencia, aleatorio simple y estratificado) frente al parámetro poblacional de ingresos (1330.72 USD). Se comprobó que tomar una muestra por conveniencia omite ciudades enteras provocando un sesgo de selección claro, por lo que para tomar decisiones la gerencia debe exigir muestreos estratificados y representativos de todas las ciudades.

## Parte A — Clasificación de las variables

*Completa la tabla usando el **criterio** (¿tiene sentido la mitad del valor? ¿hay orden entre las categorías?), no el ejemplo. En «verificación» anota qué mostró el código: dtype, número de valores únicos y, si declaraste el orden, cómo lo comprobaste.*

| Variable | Tipo | Justificación (criterio) | Verificación en código |
|---|---|---|---|
| `cliente_id` | Categórica Nominal | Es un código identificador único por cliente; no tiene sentido hacer operaciones matemáticas ni tiene orden real. | `dtype = object`, 12 valores únicos (`C001` a `C012`). |
| `edad` | Numérica Discreta | Variable cuantitativa que describe los años cumplidos registrados en números enteros. | `dtype = int64`, 12 valores únicos. |
| `numero_hijos` | Numérica Discreta | Cantidad contable de personas; no tiene sentido hablar de "la mitad" de un hijo, por lo que solo toma enteros. | `dtype = int64`, 5 valores únicos (`0, 1, 2, 3, 4`). |
| `genero` | Categórica Nominal | No existe ningún orden o jerarquía entre las opciones y presenta 3 categorías en la base. | `dtype = object`, 3 categorías (`Femenino`, `Masculino`, `Otro`). |
| `nivel_educacion` | Categórica Ordinal | Existe una secuencia y jerarquía clara que va desde Primaria hasta Posgrado. | `dtype = object` -> `CategoricalDtype` con `.min() = Primaria` y `.max() = Posgrado`. |
| `ciudad` | Categórica Nominal | Describe únicamente la ubicación geográfica del cliente sin ninguna jerarquía. | `dtype = object`, 3 categorías (`Quito`, `Guayaquil`, `Cuenca`). |
| `ingresos_mensuales` | Numérica Continua | Representa el ingreso monetario en dólares de una persona; admite promedios y valores decimales. | `dtype = int64`, 12 valores únicos (`760` a `2450`). |
| `satisfaccion` | Categórica Ordinal | Mide un nivel mediante una jerarquía del 1 (menos satisfecho) al 5 (más satisfecho). | `dtype = int64` -> `CategoricalDtype` con `.min() = 2` y `.max() = 5`. |

**P1.** ¿Qué te dijo el dtype de `satisfaccion` y por qué no basta para clasificarla? ¿Qué pasa si una ordinal queda como texto sin declarar el orden?

Para Python, la variable `satisfaccion` devolvió un tipo entero (`int64`), pero eso no basta para clasificarla porque el software no conoce el contexto estadístico: un nivel 4 de satisfacción no significa estar "el doble de satisfecho" que un nivel 2, ya que solo son etiquetas de orden. Por otro lado, si una variable ordinal como `nivel_educacion` se deja como texto sin declarar su orden con `CategoricalDtype`, Python la ordena alfabéticamente colocando `"Posgrado"` antes que `"Primaria"`, y si al declarar los niveles escribimos mal una categoría (como `"Postgrado"` con `t`), esos datos se vuelven `NaN` silenciosamente.

## Parte B — Población vs. muestra

### La población y su parámetro

| Concepto | Valor |
|---|---|
| Población (N) | 400 clientes |
| Variable de interés | Ingresos mensuales (`ingresos_mensuales`) |
| Parámetro poblacional (media) | 1330.72 USD |

*¿Por qué se muestrea si aquí el parámetro existe y se calculó?*

Porque en este laboratorio tenemos a la mano los 400 registros de `clientes_400.csv`, lo que nos permite usar el parámetro real como espejo para evaluar qué tan bien funciona cada método de muestreo. En la vida real, consultar a toda la población suele ser muy costoso o imposible porque son miles o millones de clientes.

### Los tres diseños

*Corre tu script y registra las salidas. La composición por ciudad es la que delata el sesgo: anótala tal como salió.*

| Diseño | n | Media estimada | Composición por ciudad | Diferencia vs. parámetro |
|---|---|---|---|---|
| Conveniencia (`head`) | 20 | 1348.50 USD | Quito: 20, Guayaquil: 0, Cuenca: 0 | +17.78 USD |
| Aleatorio simple (`sample`) | 20 | 1336.00 USD | Cuenca: 8, Quito: 8, Guayaquil: 4 | +5.28 USD |
| Estratificado (`groupby` + 5 %) | 20 | 1287.50 USD | Quito: 10, Guayaquil: 6, Cuenca: 4 | -43.22 USD |

*Anota también las medias de 5 sorteos distintos del aleatorio simple (semillas 1 a 5):*

| Semilla | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Media | 1336.00 USD | 1405.00 USD | 1235.00 USD | 1491.50 USD | 1288.00 USD |

**P2.** ¿Cuánto vale el parámetro poblacional y cuánto estimó cada diseño? ¿Cuál quedó más cerca y por qué eso **no** alcanza para declarar un ganador?

El parámetro poblacional es 1330.72 USD y las estimaciones fueron: Conveniencia 1348.50 USD, Aleatorio simple (semilla 1) 1336.00 USD y Estratificado (semilla 1) 1287.50 USD. El aleatorio simple quedó más cerca con una diferencia de +5.28 USD, pero no es correcto declararlo ganador solo por ese número ya que fue producto de la suerte en la primera semilla; al probar otras semillas como la 4, la estimación se alejó hasta 1491.50 USD (+160.78 USD).

**P3.** La muestra por conveniencia también tiene 20 filas. ¿Qué defecto tiene y cómo lo delatas en la composición por ciudad? ¿Se corrige tomando más filas?

Tiene un sesgo de selección claro, el cual se delata al ver que el 100 % de los clientes seleccionados son de Quito (`Quito: 20`) omitiendo por completo a Guayaquil y Cuenca, debido a que el archivo está ordenado por ciudad. No se corrige tomando más filas (como 50 o 100), porque las primeras 200 filas siguen siendo solo de Quito; solo se corrige cambiando a un método probabilístico.

**P4.** El sorteo aleatorio dio un número distinto en cada semilla. ¿Qué implica esa variabilidad para una empresa que va a decidir con un solo estudio?

Implica que con una muestra pequeña ($n=20$) existe alta incertidumbre y error de muestreo, pues entre la semilla 3 (1235.00 USD) y la semilla 4 (1491.50 USD) hay más de 250 USD de diferencia. En un negocio real no se debe decidir confiando a ciegas en un único número puntual al azar sin considerar el margen de error o usar una muestra más grande.

**P5.** ¿Cuándo usarías cada diseño en la práctica? Relaciona cada uno con un uso real (piloto, subgrupos que deben aparecer, conclusión sobre toda la población).

* **Muestreo por Conveniencia:** Útil únicamente para hacer pruebas piloto rápidas de código o validar el formato de las columnas, nunca para estimaciones reales de negocio.
* **Muestreo Aleatorio Simple:** Útil para sacar conclusiones generales cuando la población total está disponible en un listado y es homogénea.
* **Muestreo Estratificado:** Útil cuando existen subgrupos o estratos clave (como las ciudades Quito, Guayaquil y Cuenca) que deben aparecer obligatoriamente con su proporción exacta en la muestra.

## Conclusiones

* Aprendí que el tipo de dato que devuelve Python (`dtype`) es solo una guía técnica y no reemplaza el criterio estadístico, como en `satisfaccion`, donde un número entero representa en realidad una escala ordinal.
* Comprobé que el diseño de muestreo es lo que evita el sesgo de selección: tomar los primeros datos con `head(20)` dejó fuera a dos ciudades enteras, mientras que el muestreo estratificado garantizó la proporción exacta de Quito (10), Guayaquil (6) y Cuenca (4).
* Como recomendación para la gerencia de RetailAndina S.A., no se deben aprobar decisiones basadas en muestras por conveniencia ni en una sola estimación pequeña sin evaluar su variabilidad; se debe emplear muestreo estratificado por ciudad y aumentar el tamaño de muestra.
---
layout: post
title: The CIS data with R.
---

Hello and welcome to the real world, in where, under many circumstance, we can't find order in the data. Chaos! Now, we don't live in the `.csv` comfort zone. Organizations as the _Centro de Investigaciones Sociológicas_ will bring us its data in HTML tables or in particular formats such PCAXIS.

En primer lugar, activamos un paquete del R, `rvest`, parte del _Hadleyverse_^[Cualquiera que haya tocado el R sabe que éste es casi inseparable de las soluciones ingeniadas por Hadley Wickham: http://adolfoalvarez.cl/the-hitchhikers-guide-to-the-hadleyverse/]. La página a la cual le vamos a sacar los datos es ésta: <http://www.cis.es/cis/export/sites/default/-Archivos/Indicadores/documentos_html/sA306010040.html>

¿Y de qué van nuestros datos? Son los de autoubicación ideológica, esto es, dónde te ubicas ideológicamente en una escala de 1 al 10 (algo discutible, por cierto, y documentado en la propia metodología: una escala así tiende a sesgar la orientación hacia la izquierda, algo que es más improbable que pase en una escala de 0 al 10^[Para más detalles, consultar la página de metodología del CIS: http://www.cis.es/cis/opencms/ES/11_barometros/metodologia.html])

Allá vamos. Lo que hacemos, básicamente, es coger el código HTML de una web y luego, buscar entre los nodos (que pueden ser etiquetas, clases e identificadores, toda esta pesca). En casos simples como éste, una estrategia viable es leer el código fuente de la web y encontrar el identificador deseado (como la tabla del CIS va dentro de otra tabla, se hace imprescindible pillarla por identificador; de lo contrario, puede haber errores).

```{r}
library(rvest)
library(dplyr)
scrap <- html("http://www.cis.es/cis/export/sites/default/-Archivos/Indicadores/documentos_html/sA306010040.html")
df <- data.frame(html_table(html_nodes(scrap, ".dataframe")))
glimpse(df)
```

Podemos notar cómo R, por defecto, da un nombre del estilo Var.1 a la primera columna, que originalmente carece de nombre. Cambiarlo es algo trivial. Por cierto, hay que notar lo útil que es la función `glimpse`, dentro de la librería `dplyr`. Nos permite ver rápidamente de qué está hecha nuestra tabla, una operación muy recomendable después de extraer datos de nuestra web.

Ahora, ya que en teoría tenemos la tabla, probamos a graficar, empezando por el sistema simple de gráficos:

```{r}
(tsIzq <- df[1,])
tsIzq[1] <- NULL
ts.plot(tsIzq)
```

Oh, una gráfica vacía. Nos damos cuenta de que nuestros datos necesitan un poco de limpieza. En primer lugar, extraemos los nombres y eliminamos la columna. También, a mi juicio, elimino la variable _N_, por considerarla irrelevante para el análisis que quiero hacer.

```{r}
namesdf <- df$Var.1
df$Var.1 <- NULL
N <- df[8,]
df <- df[-8,]
glimpse(df)
```

Vemos que en septiembre del 2013 hay valores perdidos. Hay varias estrategias para abordar este punto (nunca mejor dicho). En primer lugar, ser honestos e incluir los valores perdidos.

```{r}
df$sep13[df$sep13=="."] <- NA
df$sep13
```

En segundo lugar, si queremos rellenar esto de significado, podemos imputar. Empecemos por una media simple de los doce meses anteriores.


```
# Para una variable del CIS en concreto

library(rvest)
scrap <- html("http://www.cis.es/cis/opencms/-Archivos/Indicadores/documentos_html/IndiAubid.html")
# al parecer, todas las tablas acaban identificándose por la clase .dataframe. Eso facilita las cosas
autoubMed <- data.frame(html_table(html_nodes(scrap, ".dataframe")))
# Vamos al dato que nos interesa
ind <- autoubMed [1,]
ind[1] # hay un valor que no coincide, entonces...
ind[1] <- NULL
# Ahora tenemos clara una cosa: es una serie temporal, pero en primer lugar la forzaremos a ser una matriz
ind2 <- as.numeric(as.matrix(ind))
# Usamos una frecuencia de 11 porque no se hacen los barómetros en agosto.
ts.ind <- ts(ind2, start=c(1996, 1), end=c(2015, 6), frequency=11)
# Ahora podemos graficar:
ts.plot(ts.ind, type="l", main="Evolución de la autoubicación ideológica \nmedia en España", sub="Fuente: Barómetros del CIS", col="red")
# Y con GGPlot2
library(ggplot2)
library(ggfortify) # necesario para que ggplot2 sepa cómo tratar las series temporales
autoplot(ts.ind, ts.colour=('dodgerblue3')) + ggtitle("Autoubicación ideológica media en España (1996-2015) según los barómetros del CIS")

# Beautiful predictions!

library(forecast)
ts.indA <- auto.arima(ts.ind)
ind.forecast <- forecast(ts.indA, level = c(95), h = 50) # por defecto, luego se puede ir revisando
autoplot(ind.forecast, predict.linetype = 'dashed') # ahí vemos un escenario problemático: hay mucha estabilización en las tendencias en comparación con los números anteriores, en torno a ~4.6
```

---
layout: post
title: The CIS data with R.
---

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

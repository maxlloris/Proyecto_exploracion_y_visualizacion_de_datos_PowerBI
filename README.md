
   # **Práctica exploración y visualización de datos**

Exploración y visualización de Dataset de E-Commerce


## Origen de los datos

Desarrollo la práctica y el análisis de los datos sobre un Dataset que recoge las transacciones de un **E-Commerce**.
**Link:** [Dataset E-Commerce](https://www.kaggle.com/datasets/carrie1/ecommerce-data/data/)


En base a estos datos y utilizando distintas funciones de DAX, he procedido a la obtención de más columnas y medidas calculadas que nos permitirán, más adelante, plantear distintos KPI a los cuales daré respuesta en los distintos Dashboards.


## Transformación manual de los datos

Las columnas InvoiceNO, StockCODE, Description los dejamos tal y como nos vienen de origen.
Para la columna Quantity le indicamos que se trata de un entero.
La columna InvoiceData le cambiamos el tipo y seleccionamos la configuración regional de origen.
Para la columna de UnitPrice, sustituimos el punto que define el decimal y lo sustituimos por coma para poder hacer la conversión a número decimal.
El resto de columnas las dejamos tal y como vienen, salvo la columna de **Country** sobre la cual aplicamos un filtro para excluir al Reino Unido por ciertos problemas que daban los registros que contenían este país, tal y como se indicaba en las especificaciones de la data set.


## Obtención de más columnas

### Fecha

Las primeras columnas adicionales que he obtenido ha sido la de las siguientes:

Hour

```
Hour = HOUR('Invoice data'[InvoiceDate])
```

Hour Group
```
Hour Group = IF ('Invoice data'[Hour] < 12, "AM", "PM")
```

Day
```
Day = WEEKDAY('Invoice data'[InvoiceDate], 2)
```
Nota: el 2 responde al segundo parámetro de la función WEEKDAY en el cual hay que especificar cual es el primer dia de la semana.


Day of Week
```
Day of Week = FORMAT( 'Invoice data'[InvoiceDate], "DDDD")
```
Nota: el segundo parámetro “DDDD” hace referencia a la forma en la que queremos visualizar el dia de la semana. En este caso, esta es la sintaxis para el nombre completo del día, Lunes por ejemplo.


### Categoría del producto

Con esta categoría establecemos en función del código del producto, si se trata de un producto nacional o de origen importado.
Para este ejercicio se define como producto importado todo aquel que tenga un StockCode con un LEN superior a 5.
Para mayor clasificación se definen otro criterio que indicara el origen de aquellos productos que han sido importados. "Imported from Europe" o "Imported Worldwide" en función de la última letra del StockCode

Con estos dos criterios calculamos y obtenemos las siguientes dos columnas:

Stock Category
```
Stock Category = IF(

RIGHT('Invoice data'[StockCode], 1) = "A" ||

RIGHT('Invoice data'[StockCode], 1) = "B" ||

RIGHT('Invoice data'[StockCode], 1) = "C",

"Imported from Europe",

IF(

RIGHT('Invoice data'[StockCode], 1) = "D" ||

RIGHT('Invoice data'[StockCode], 1) = "E" ||

RIGHT('Invoice data'[StockCode], 1) = "F" ||

RIGHT('Invoice data'[StockCode], 1) = "G",

"Imported Worldwide", "National Product"
))
```
Nota: Con la función RIGHT aplicada sobre una columna e indicando la posición del elemento obtenemos el primer elemento de cada StockCode y comprobamos si cumple la condición booleana.


Stock Category 2
```
Stock Category 2 = IF (LEN('Invoice data'[StockCode]) > 5, "Imported Product", "National Product")
```

### Categoría por género

Para este ejercicio establecemos la condición que determina el género del cliente en función de que el último elemento del CustomerID sea par o impar
Customer Gener
```
Customer Gener = IF(MOD((CONVERT(RIGHT('Invoice data'[InvoiceNo], 1), INTEGER)), 2) = 0, "Female", "Male")
```
 
### Agrupación de países

Agrupamos a los países en dos categorías. Aquellos que tiene mas de un 10% de las ventas totales los incluimos en la categoría “Countries High amount %” y el resto de países con porcentajes inferiores en la categoría “Countries Low amount %”. Para ello utilizamos la función de agrupación de propio Power BI


## Creación de tabla para medidas adicionales

Para llevar a cabo medidas adicionales necesarias para contestar a las nuevas columnascreo una nueva tabla DAX medidas para poder recogerlas.
Algunas de estas medidas, se han obtenido y aplicado sobre gráficos que finalmente muestro y otras finalmente no se han utilizado, pero aun así las mantengo por que pueden ser requeridas en cualquier momento si se amplía el alcance de los Dashboards.
```
Customer Count = DISTINCTCOUNT('Invoice data'[CustomerID])

Customer Count % = [Customer Count] / CALCULATE([Customer Count], ALL('Invoice data'))

Invoice Count = DISTINCTCOUNT('Invoice data'[InvoiceNo])

Invoice Count = DISTINCTCOUNT('Invoice data'[InvoiceNo])

National Product $ = CALCULATE([Total Amount], 'Invoice data'[Stock Category] = "National Product")

National Producto % = [National Product $] / [Total Amount]

Total Amount = SUMX('Invoice data', [Quantity] * [UnitPrice])

Total amount % = [Total Amount] / CALCULATE([Total Amount], ALL('Invoice data'))

Unit Price Average = AVERAGE('Invoice data'[UnitPrice])
```
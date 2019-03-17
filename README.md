# Parte 1: Web Scrapping
## 1.1 Creación de paraderos
Lo primero que hacemos es encontrar la información relativa a cada uno de los paraderos. Para eso, se utilizaron las funciones del material de clases para transformar el código fuente de páginas web a formato string. 

Con el código fuente en formato string se usaron varias funciones para encontrar la información que se necesitaba: 


1.   **Encuentra tabla:** Esta función recibe un string y separa todas las tablas que encuentre en ese string. Para eso busca donde dice  "div class="twelve columns" y corta hasta que encuentre otro "div" 

2.   **Encuentra fake:** Dada una tabla busca si es que en alguna parte está el string "fake" o "Fake". Si lo encuentra, retorna "True"

3. **Próxima URL:** Dada un string, busca dónde está el link que lleve a la siguiente página. Para eso se busca los textos que estén entre "a" y luego se busca "href".

4. **From string to list:** Esta función recibe como input una tabla en formato string y devuelve una lista de listas con toda la información de la tabla. En este caso [paradero, Nservicios, buses].

Utilizando estas funciones se hace el scrapping para las 3 páginas de trygger milenio. Se guardan todos los datos encontrados en una lista llamada "Datos". Posteriormente se pasa esta lista a formato ".csv" utilizando la librería csv. 


## 1.2 Obtención de datos para cada paradero
Para esta parte se había creado la función para buscar la información de todos los paraderos en la API de Transantiago; no obstante, debido a que era muy lento hacer esta consulta, y siguiendo las instrucciones del issue presente en GitHub se hace la consulta sólo para un servicio.

Este paradero queda a criterio del usuario, quien tiene que entregar la información cuando aparece "Ingrese consulta". Luego de esto, se hace el web scrapping en la API tomando los datos de todos los paraderos que contengan a ese servicio. Esto se hace transformando el csv de paraderos a formato dataframe y filtrando para el servicio que necesitamos. 

La consulta se hace 3 veces, cada 5 minutos cada una.

# Parte 2: Utilización de GeoPandas
## 2.1 Archivos shape
En segundo lugar, se cargan los recursos correspondientes al shape de la ciudad de santiago, shape de trazados y shape de paraderos
(especificamente, para esto se creo un repositorio publico en donde se almacenaron todos archivos obtenidos desde https://github.com/carnby/carto-en-python para los shape de santiago, ademas de los proporcionados en Syllabus, esto por comodidad ya que de esta manera no se requieren carpetas locales adicionales ->[LINK REPO](https://github.com/iasantibanez/Archivos_L06)
## 2.2 Ubicacion paraderos y trazados
Posteriormente se filtran los paraderos segun el servicio consultado, y se filtran los recorridos del mismo en cada uno de los shape,
se procede a graficar esta información preliminar.
## 2.2 Ubicacion buses con toda la informacion anterior.
Para ubicar los buses, se realizo el supuesto escogiendo el paradero que estuviera mas cercano segun cada patente para así obtener mejor precisión del bus, posteriormente se calcula el sentido correcto al que pertenece(trazado).

Sin perder generalidad, según tiempo estimado se procedio a generar un buffer de radio:"distancia al paradero" (info sacado del tiempo_esperado.csv), que intersectará con la el linestring(trazado del recorrido). para esto se transformo en un geodataframe auxiliar la misma información pero en formato de poligono (es decir se aplico buffer al linestring para encontrar interseccion entre poligonos mediante el comando overlay de geopandas). por ultimo, mediante la intersección se obtiene la ubicación aproximada del bus.


(SE INTENTO UNA SEGUNDA METODOLOGíA, que recorria los puntos del linestring (trazado) e iria acumulando la distancia hasta la distancia al paradero, sin embargo se optó por la primera explicada dado que genera resultados de manera mas rápida y con menos supuestos).

# Parte 3: Consultas
## 3.1 Avance total
Para esta consulta lo que se hizo fue buscar todas las observaciones que se tenía de un bus en particular. Es posible apreciar de que estas observaciones provienen de distintos paraderos; no obstante, si dos paraderos miden la distancia del bus en dos instantes consecutivos, el avance debería ser relativamente similar. Esto, obviando el tema de que las rutas no son necesariamente lineales y esto puedo formar diferencias.

Lo que se hizo fue tomar los "avances" medidos por cada uno de los paraderos y promediarlos. Es un supuesto bastante sencillo, pero los resultados que entrega la función son bastante buenos. 

Si un bus no posee más que una observación, entonces la función retorna un aviso diciendo que no es posible determinar el avance ya que hay muy pocos datos.

## 3.2 Buses llegando al paradero
Lo que se entendió de esta función es que hay que retornar todos los buses que están a una cierta distancia del paradero, pero *que pasan por el paradero*. Osea no tiene sentido mostrar un bus que está a 1m del paradero si es que no pasa por este. Este es el supuesto más fuerte que se hizo para esta parte.
Lo que se hizo fue simplemente filtrar el dataframe para el paradero y distancia requeridas. El output de la función es un dataframe. 

## 3.3 Distancia entre buses

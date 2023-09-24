Índice CHF: Análisis del CHF en los países de Costa Rica y Nicaragua
================
true
today

# Introducción

En este trabajo, se aborda la creación de un índice esencial para
comprender la climatología de huracanes en las regiones de Costa Rica y
Nicaragua: el CHF (Costal Hurricane Frequency), implementado en el
articulo de Karthik Balaguru, “Increased U.S. coastal hurricane risk
under climate change”. En este estudio se emplea una base de datos de
HURDAT (Hurricane Database) para obtener una estimación del CHF para las
regiones previamente mencionadas. Es importante destacar que, si bien se
utiliza la técnica CHF como punto de partida, se han realizado ajustes
para adaptarla a las características que tiene la base de datos del
HURDAT.

El CHF se define como la cantidad de ubicaciones de trayectorias de
huracanes que cruzan sobre tierra durante un periodo de seis horas por
grado cuadrado por año. Para su cálculo, se consideran todas las
ubicaciones de tormentas cuya intensidad supera un umbral específico. El
cual tiene la siguiente ecuación:

$$
CHF = n\frac{l}{TimeStep*v}
$$

En donde, $$
n: \textrm{es el número de trayectorias de huracanes que pasan a través de un grado cuadrado}
$$ $$
l: \textrm{la longitud de la cuadrícula}
$$

$$
v: \textrm{la velocidad de traslación}
$$

$$
TimeStep: \textrm{es el paso de tiempo}
$$

## Lenguaje de programación

Para la creación de dicho índice se utilizará el lenguaje de
programación de Python, con uso de las librerías de manipulación de
datos que tiene esta; mientras que el análisis gráfico se hará uso del
lenguaje R, puesto que la librería de `ggplot` es estéticamente mejor.
Para pode utilizar ambos lenguajes de programación simultaneamente se
utilizó la librería llamada `reticulate`.

Las siguientes librerías serán las que se van a utilizar para el
lenguaje de Python:

- **NumPy:** Libreria para realizar cálculos numéricos en Python. Es
  ampliamente utilizada para operaciones matemáticas y de manipulación
  de datos.

- **Pandas:** Pandas es una biblioteca que proporciona estructuras de
  datos y herramientas para el análisis de datos. Es especialmente útil
  para la manipulación y exploración de datos tabulares.

- **GeoPandas:** GeoPandas es una extensión de Pandas que agrega
  capacidades geoespaciales. Permite trabajar con datos geoespaciales,
  como GeoDataFrames, y realizar análisis espaciales.

- **OS:** El módulo `os` proporciona funciones para interactuar con el
  sistema operativo, como la manipulación de rutas de archivos y
  directorios.

- **Shapely:** Shapely es una biblioteca que se utiliza para el
  procesamiento y análisis de geometría plana. Permite trabajar con
  objetos geométricos como puntos, líneas y polígonos.

- **pyogrio:** pyogrio es una biblioteca que se utiliza para leer y
  escribir datos geoespaciales en formatos como GeoJSON y Shapefile.
  Facilita la manipulación de datos geoespaciales.

``` python
import numpy as np
import pandas as pd
import geopandas as gpd
import os 
import shapely
from pyogrio import read_dataframe
from shapely.geometry import Polygon, Point
```

Las siguientes librerias seran las que se van a utilizar para el
lenguaje de R:

- **Tidyverse:** Conunto de librerias que contiene a algunas como
  `ggplot` y `dplyr`, entre otras… Principalmente se utilizaran ggplot y
  dplyr.

- **ggthemes:**

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.2     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.2     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.1     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(ggthemes)
library(sf)
```

    ## Linking to GEOS 3.11.0, GDAL 3.5.3, PROJ 9.1.0; sf_use_s2() is TRUE

``` r
library(geojsonsf)
library(viridis)
```

    ## Loading required package: viridisLite

# **Análsis**

## **Base de datos**

Tal como se mencionó en la introducción, se procederá a importar la base
de datos del HURDAT (Hurricane Database), en conjunto con los archivos
de shape (shapefiles) que representan los límites geográficos de los
países de Costa Rica y Nicaragua.

``` python
#Importamos las bases de datos
hurdat = pd.read_csv("/Users/manriquecamacho/Library/CloudStorage/OneDrive-UniversidaddeCostaRica/Universidad de Costa Rica/HORAS/CIGEFI/Datos Manrique/DATOS HURDAT copy/hurdat2daily.csv")
cr = read_dataframe("/Users/manriquecamacho/Library/CloudStorage/OneDrive-UniversidaddeCostaRica/Universidad de Costa Rica/HORAS/CIGEFI/Datos Manrique/Shape_CR_NIC/CR/CR.shp")
nic = read_dataframe("/Users/manriquecamacho/Library/CloudStorage/OneDrive-UniversidaddeCostaRica/Universidad de Costa Rica/HORAS/CIGEFI/Datos Manrique/Shape_CR_NIC/NIC/NIC.shp")
```

## **Análisis Descriptivo**

En esta sección, se llevará a cabo un análisis descriptivo utilizando la
base de datos HURDAT (Hurricane Database), que contiene información
detallada sobre la velocidad, nombre, latitud, longitud, fecha (en
año-mes-dia) y radio de cada huracán registrado. Este análisis inicial
se centrará en la exploración y comprensión de las características
fundamentales de los huracanes registrados en la región, sin considerar
aún la creación de los índices CHF. Dentro de esta sub-sección se
llevaran a cabo los siguientes puntos:

1.  Se analizará y se hará una manipulación en la base de datos del
    HURDAT para poder tener una mayor utilidad en los futuros análisis.

2.  Se examinará la distribución de la velocidad de los huracanes y su
    variabilidad a lo largo de los años y meses.

3.  Se delimitará la atención a los huracanes que afectaron directamente
    o estuvieron cerca de las costas de Costa Rica y Nicaragua. Esto
    permitirá realizar un análisis más detallado de las características
    de los huracanes que tienen un impacto potencial en esta región, lo
    que incluye su intensidad, tamaño y trayectoria.

### **Parte 1: Manipulación**

Primeramente se observa como está organizada la base de datos del HURDAT

``` python
hurdat.head()
```

    ##        ATCF     Name  rows         dat   Lat  ...   64NE   64SE   64SW   64NW    Rad
    ## 0  AL011851  UNNAMED    14  1851-06-24  28.0  ... -999.0 -999.0 -999.0 -999.0 -999.0
    ## 1  AL011851  UNNAMED    14  1851-06-25  28.1  ... -999.0 -999.0 -999.0 -999.0 -999.0
    ## 2  AL011851  UNNAMED    14  1851-06-26  28.6  ... -999.0 -999.0 -999.0 -999.0 -999.0
    ## 3  AL011851  UNNAMED    14  1851-06-27  30.2  ... -999.0 -999.0 -999.0 -999.0 -999.0
    ## 4  AL021851  UNNAMED     1  1851-07-05  22.2  ... -999.0 -999.0 -999.0 -999.0 -999.0
    ## 
    ## [5 rows x 21 columns]

Como se observa y se menciona anteriormente en la introducción de la
sección “Análisis Descriptivo” la columna dat, que se refiere a la fecha
de ese huracán, está agrupada. Por lo cual se decide dividir dicha
columna en tres columnas, las cuales serian las siguientes: Año, Mes y
Día. Esto con el fin de poder analizar el comportamiento a través de los
años de los huracanes

``` python
hurdat[["anio","mes","dia"]] = hurdat["dat"].str.split('-',expand=True)

hurdat.drop(columns= 'dat', inplace= True)
```

``` python
hurdat = hurdat[hurdat["MaxW"]>50]
```

Seguidamente, se crea un DataFrame el cual contiene a los paises de
Nicaragua y Costa Rica concatenados. Ademas de pasar la base de datos
del HURDAT, a un dataframe geoespacial.

``` python
cr_nic = pd.concat([cr,nic]) #Concatenamos ambas bases
cr_nic["Pais"] = ["CR","NIC"] #Designamos cual geometría le pertenece a que país
gdf_hurdat = hurdat.copy()
gdf_hurdat =  gpd.GeoDataFrame(gdf_hurdat,
                                geometry=gpd.points_from_xy(gdf_hurdat.Lon, gdf_hurdat.Lat), 
                                crs="EPSG:4326") #Base del Hurdat con formato geoespacial
```

Ya teniendo a Costa Rica y Nicaragua agrupados, se procede a crear una
nueva base de datos en la cual se filtró la base del HURDAT para que
solo aparezcan los huracanes con las coordenadas cercanas a los paises
de Nicaragua y Costa Rica.

``` python
minx, miny, maxx, maxy = cr_nic.total_bounds #Limites de los paises de Nicaragua y Costa Rica
hurdat_cr_nic = gdf_hurdat[((gdf_hurdat["Lon"]<= maxx)&
                            (gdf_hurdat["Lon"]>= minx))&
                            ((gdf_hurdat["Lat"]<= maxy)&
                            (gdf_hurdat["Lat"]>= miny))] #Delimitación de la zona
```

### **Análisis Gráfico Global de la Velocidad**

Primero, se utilizará la base de datos completa de HURDAT, que abarca
todos los huracanes en todo el mundo durante los años de 1851 a 2021.
Esto permitirá observar y analizar la variabilidad de las velocidades a
lo largo de los años y comprender los diversos comportamientos de estos
fenómenos. Posteriormente, se llevará a cabo un análisis similar, pero
enfocado exclusivamente en los huracanes que se acercaron a las costas
de Costa Rica y Nicaragua.

``` r
df2 = py$hurdat  %>% 
  mutate(anio_agrupados = case_when(
    anio<=1885 ~ '1851-1885',
    (anio>1885)&(anio<=1919) ~'1886-1919',
    (anio>1919)&(anio<=1954) ~ '1920-1954',
    (anio>1954)&(anio<=1988) ~ '1955-1988',
    (anio>1989) ~ '1989-2023',
  )) %>% 
  drop_na() %>% 
  group_by(anio_agrupados) %>% 
  summarise(prom.anio = mean(MaxW))
  
py$hurdat %>% 
  mutate(anio_agrupados = case_when(
    anio<=1885 ~ '1851-1885',
    (anio>1885)&(anio<=1919) ~'1886-1919',
    (anio>1919)&(anio<=1954) ~ '1920-1954',
    (anio>1954)&(anio<=1988) ~ '1955-1988',
    (anio>1989) ~ '1989-2023',
  )) %>% 
  drop_na() %>% 
  ggplot(aes(x = MaxW, fill = anio_agrupados))+
    geom_histogram()+
    geom_vline(data = df2, aes(xintercept = prom.anio),
             linetype = "dashed",
             color= "black")+
    geom_label(data = df2,
              mapping = aes(x = prom.anio+19, y = 200, label = paste("Promedio",round(prom.anio))),
              label.padding = unit(0.15, "lines"),
              fill = 'white')+
    facet_grid(~anio_agrupados,scales ="free_x")+
  scale_fill_tableau()+
    labs(title = "Frecuencia de Velocidad de Huracanes", y = "Frecuencia", x = "Velocidad",subtitle = "Filtrado por años desde 1851 a 2023",caption = "Elaboracion propia: Manrique Camacho P.")+
  scale_x_continuous(breaks = c(20,40,60,80,100,120))+
  guides(fill='none')+
  theme_fivethirtyeight()+
  theme(strip.text = element_text(face = "bold"))
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Reporte_CHF_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

Como se presencia en el “Gráfico 1: Frecuencias de Velocidades de los
Huracanes”, se observa una relativa estabilidad en la velocidad de los
huracanes a lo largo de los años, con la excepción del período entre
1851 y 1885 a través de todo del mundo. Este cambio puede atribuirse
posiblemente a limitaciones en los instrumentos de medición de la época,
ya que se identifica un pico en el subgráfico que sugiere la existencia
de una tendencia hacia ese valor en los datos recopilados durante ese
intervalo de tiempo. Posteriormente, a partir de 1885, las velocidades
de los huracanes tienen una distribución similar, con características
que se asemejan a una distribución gamma, marcada por su asimetría.
Además como se puede observar, los promedios son muy similares alrededor
de 75, por lo cual que utilizar el promedio para el futuro cálculo del
CHF no es tan malo, esto se mencionará a más detalle posteriormente. Sin
embargo cabe mencionar que se está tratando con distribuciones
asimétricas por lo cual sería mas pertinente trabajar con las medianas.

Luego, se procede a la representación gráfica de las frecuencias de
velocidad desde 1851 hasta 2023 en la región cercana a Costa Rica y
Nicaragua. Este gráfico tiene como finalidad brindar una comprensión más
precisa sobre cómo se distribuyen las velocidades en las inmediaciones
de estos países. El objetivo subyacente es determinar si esta
distribución sigue una pauta similar a la que se observa en la
distribución de velocidades de huracanes a nivel global, tal como se
presenta en el “Gráfico 1: Frecuencias de Velocidades de los Huracanes”.

``` python
hurdat_grafico_cr_nic = hurdat_cr_nic.copy()
hurdat_grafico_cr_nic = hurdat_cr_nic.drop('geometry',axis=1)
```

``` r
py$hurdat_grafico_cr_nic %>% 
  ggplot(aes(x = MaxW))+
  geom_histogram(fill = "#cf3e53", alpha = 0.8, color = "black",binwidth = 10)+
    geom_vline(aes(xintercept = mean(MaxW)),
             linetype = "dashed",
             color= "black")+
    geom_label(mapping = aes(x = mean(MaxW)+5.5, y = 7, label = paste("Promedio",round(mean(MaxW)))),
              label.padding = unit(0.15, "lines"))+
    labs(title = "Frecuencia de Velocidad de Huracanes en Costa Rica y Nicaragua", y = "Frecuencia", x = "Velocidad",subtitle = "Desde 1851 hasta 2023",caption = "Elaboracion propia: Manrique Camacho P.")+
  scale_x_continuous(breaks = c(20,40,60,80,100,120))+
  scale_y_continuous(breaks = c(1,2,3,4,5,6,7,8,9))+
  guides(fill='none')+
  theme_fivethirtyeight()+
  theme(strip.text = element_text(face = "bold"))
```

![](Reporte_CHF_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

La distribución de velocidades de los huracanes que han afectado las
proximidades de Costa Rica y Nicaragua muestra una notable similitud con
las velocidades observadas en el “Gráfico 1: Frecuencias de Velocidades
de los Huracanes”. Esto indica que las velocidades de los huracanes no
exhiben un comportamiento irregular cuando atraviesan estas regiones
mencionadas. Es importante destacar que la cantidad total de huracanes
que afectan estas zonas es bastante reducida, como se puede apreciar en
los datos. Además, el promedio es un poco mayor que el del gráfico
anterior.

## **Análisis del Indicador CHF**

En esta subsección se realizaran los siguientes puntos:

1.  Creación del indicador CHF
2.  Graficación del indicador CHF

### **Creación del Indicador CHF**

Para la creación del indicador CHF, el cual aparece en el artículo
científico *Increased U.S. coastal hurricane risk under climate change*
de Karthik Balaguru se debe tomar en cuenta varios aspectos. La formula
mencionada en la introducción la cual es la siguiente:

$$
CHF = n\frac{l}{TimeStep*v}
$$

En donde, $$
n: \textrm{es el número de trayectorias de huracanes que pasan a través de un grado cuadrado}
$$ $$
l: \textrm{la longitud de la cuadrícula}
$$

$$
v: \textrm{la velocidad de traslación}
$$

$$
TimeStep: \textrm{es el paso de tiempo}
$$

Debido a que se está utilizando la base de datos del HURDAT hay una
limitación en la implementación de este estimador, de la manera que lo
propone Balaguru, puesto que dicha base no tiene las mismas condiciones
propuestas en la formula. Seguidamente se brindaran las observaciones
que habran en este cálculo del CHF.

- La base del HURDAT tiene un **TimeStep** de 24 horas y no de 6 como
  sugiere Balaguru

- En el artículo de Balaguru no se especifica como se utiliza el tiempo
  de traslación, es decir **v**, por lo cual para la implementación de
  este indice se decidió tomar el promedio de la máxima velocidad (MaxW)
  por grado cuadrado.

- Por ultimo, cabe mencionar que el radio (Rad) no se utiliza en el
  estimador CHF pero es una variable que se podría contemplar incluirla
  de cierta manera en el cálculo.

#### **Creación de cuadrículas**

\\ Se prosigue a crear las cuadrículas que se van a utilizar para el
índice del CHF, cabe resaltar que se menciona en el libro de Balaguru
que las cuadrículas son grado cuadrado.

``` python
grado_cuad = 1 #Grado cuadrado = metro cuadrado

x_coords = np.arange(minx, maxx, grado_cuad)
y_coords = np.arange(miny, maxy, grado_cuad)

celdas = [] #Lista donde guardaremos las coordenadas de las celdas

for x in x_coords:
    for y in y_coords:
        poligono = Polygon([(x, y), 
        (x + grado_cuad, y), 
        (x + grado_cuad, y + grado_cuad), 
        (x, y + grado_cuad)])
        celdas.append(poligono)
        
celdas[:5] #Mostramos solo los primeros 5 resultados
```

    ## [<POLYGON ((-87.7 5.5, -86.7 5.5, -86.7 6.5, -87.7 6.5, -87.7 5.5))>, <POLYGON ((-87.7 6.5, -86.7 6.5, -86.7 7.5, -87.7 7.5, -87.7 6.5))>, <POLYGON ((-87.7 7.5, -86.7 7.5, -86.7 8.5, -87.7 8.5, -87.7 7.5))>, <POLYGON ((-87.7 8.5, -86.7 8.5, -86.7 9.5, -87.7 9.5, -87.7 8.5))>, <POLYGON ((-87.7 9.5, -86.7 9.5, -86.7 10.5, -87.7 10.5, -87.7 9.5))>]

#### **Creación del Indice CHF**

Ya teniendo las cuadrículas definidas, se prosigue a crear el indice
CHF. La formula empleada para el cálculo del CHF quedo de la siguiente
manera: $$
CHF = \frac{n_i}{6\overline{v}_{i}}
$$ En donde, $$
n_i: \textrm{es el número de huracanes que pasaron a través de la cuadricula i}
$$ $$
v_i: \textrm{velocidad máxima de traslación promedio de la cuadrícula i}
$$

``` python
grid_gdf = gpd.GeoDataFrame({'geometry': celdas}, crs=cr_nic.crs)

# Calcular la frecuencia de huracanes en cada cuadrícula
grid_gdf['frecuencia_h'] = grid_gdf['geometry'].apply(lambda geom: len(hurdat_cr_nic[hurdat_cr_nic.within(geom)]))

# Calcular el promedio de velocidad de huracanes en cada cuadrícula
grid_gdf['velocidad_promedio'] = grid_gdf['geometry'].apply(lambda geom: hurdat_cr_nic[hurdat_cr_nic.within(geom)]['MaxW'].mean())

# Sumar el promedio de velocidad a la frecuencia en un nuevo campo
grid_gdf['Indicador_CHF'] = grid_gdf['frecuencia_h']/(grid_gdf['velocidad_promedio']+24)

grid_gdf[grid_gdf["frecuencia_h"]>0].head()
```

    ##                                              geometry  ...  Indicador_CHF
    ## 5   POLYGON ((-87.69097 10.49903, -86.69097 10.499...  ...       0.013072
    ## 17  POLYGON ((-86.69097 12.49903, -85.69097 12.499...  ...       0.013158
    ## 27  POLYGON ((-85.69097 12.49903, -84.69097 12.499...  ...       0.022949
    ## 28  POLYGON ((-85.69097 13.49903, -84.69097 13.499...  ...       0.010638
    ## 29  POLYGON ((-85.69097 14.49903, -84.69097 14.499...  ...       0.010101
    ## 
    ## [5 rows x 4 columns]

Ya teniendo creado el indice CHF exportamos los datos a un documento
GeoJson para poder manipular la base y poder graficar con la librería
`sf`

``` python
documento = 'Indice_CHF.geojson'
with open(documento , 'w') as file:
    file.write(grid_gdf.to_json())
```

    ## 23013

``` python
documento = 'CR_NIC.geojson'
with open(documento , 'w') as file:
    file.write(cr_nic.to_json())
```

    ## 4757507

``` r
indice = geojson_sf("Indice_CHF.geojson")
cr_nic = geojson_sf("CR_NIC.geojson")
```

``` r
indice = indice %>% 
  mutate(Indicador_CHF = as.numeric(Indicador_CHF))
ggplot(data = cr_nic)+
  geom_sf(data = cr_nic, fill = "black",alpha=0.3)+
  geom_sf(data = indice, aes(fill = Indicador_CHF),alpha = 0.6,color = "grey50")+
  coord_sf(xlim = c(-87.47, -81.95), ylim = c(7.9, 15))+
  scale_fill_gradient(low="burlywood", high="red3", na.value="white")+
  labs(title = "Indicador Costal Hurricane Frequency",subtitle = "En los paises de Nicaragua y Costa Rica",caption = "Elaboración propia:Manrique Camacho P.",fill = "CHF")+
  theme_light()+
  theme(plot.title = element_text(face="bold",size = 10),
        plot.subtitle = element_text(size=8),
        plot.caption = element_text(size = 8))
```

![](Reporte_CHF_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

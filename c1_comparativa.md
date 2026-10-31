# C1 · Comparativa de servicios geográficos

Resumen de lo que entrega cada estándar, cómo se filtra y cómo se pagina.

| Servicio | Qué entrega | Cómo se filtra | Cómo se pagina |
|---|---|---|---|
| **WMS** | **Imágenes** de mapa ya renderizadas (PNG, JPEG, GIF, SVG…). El servidor aplica el estilo; el cliente no recibe geometrías ni atributos. | Por *lo que se dibuja*: `LAYERS`, `BBOX`, `CRS`, `WIDTH`/`HEIGHT`, `STYLES`, `TIME`. Para consultar atributos de un punto se usa `GetFeatureInfo` (`QUERY_LAYERS`, `I`, `J`). GeoServer añade `CQL_FILTER` como extensión propia. | **No se pagina**: cada petición devuelve una sola imagen. Para cubrir más territorio se piden más `BBOX` (o se usa WMTS con teselas). |
| **WFS** | **Vectores**: geometrías y atributos (GML por defecto; GeoJSON si el servidor lo ofrece, p. ej. `outputFormat=application/json` en GeoServer). | `BBOX`, `FILTER` (XML del estándar OGC Filter), `CQL_FILTER` (extensión de GeoServer), `typeNames`, `propertyName` para elegir campos. `resultType=hits` solo cuenta. | `count` + `startIndex` (WFS 2.0). En WFS 1.x se usa `maxFeatures`. Conviene ordenar (`sortBy`) para que las páginas sean estables. |
| **FeatureServer** (Esri REST) | **Vectores en JSON**: `f=json` (formato Esri: `features[].attributes` + `geometry`) o `f=geojson` (GeoJSON estándar). | `where` (expresión SQL), `geometry` + `spatialRel`, `objectIds`, `outFields`, `outSR`, `returnGeometry`. | `resultOffset` + `resultRecordCount`. El servidor impone un máximo (`maxRecordCount`) y avisa con `exceededTransferLimit` cuando quedan registros. |
| **OGC API – Features** | **Vectores en GeoJSON** (también HTML y otros formatos) por rutas REST: `/collections/{id}/items`. | `bbox`, `datetime`, parámetros por propiedad (`?campo=valor`, si la colección los declara) y `filter` (CQL2) si el servidor lo implementa. | `limit` y enlaces `next`/`prev` dentro de la respuesta; la respuesta incluye `numberMatched` y `numberReturned`. pygeoapi acepta además `offset`. |

## Peticiones de ejemplo

| Servicio | Petición |
|---|---|
| WMS | `…/WMSServer?service=WMS&version=1.3.0&request=GetMap&layers=<capa>&styles=&crs=EPSG:4326&bbox=<latMin,lonMin,latMax,lonMax>&width=800&height=600&format=image/png` (en 1.3.0 con EPSG:4326 el orden del `BBOX` es **lat,lon**; con `CRS:84` es lon,lat) |
| WFS | `…/wfs?service=WFS&version=2.0.0&request=GetFeature&typeNames=<capa>&count=3&startIndex=0` |
| FeatureServer | `https://services6.arcgis.com/Ex2fmFOuTOHRZtML/ArcGIS/rest/services/Veredas_Boyaca_2024/FeatureServer/0/query?where=1=1&outFields=*&resultRecordCount=3&resultOffset=0&f=geojson` |
| OGC API – Features | `https://demo.pygeoapi.io/master/collections/<id>/items?limit=3&offset=0&f=json` (la lista de `<id>` sale de `…/collections?f=json`) |

## Estructura del JSON: FeatureServer vs OGC API

- **FeatureServer con `f=json`**: objeto con `fields`, `spatialReference` y `features[]`, donde cada elemento trae `attributes` y `geometry` (con `rings` en polígonos). Con `f=geojson` devuelve una `FeatureCollection` estándar.
- **OGC API – Features**: siempre una `FeatureCollection` GeoJSON, con `features[]` (`type`, `id`, `geometry`, `properties`) y, además, `numberMatched`, `numberReturned`, `timeStamp` y `links` (`self`, `next`, `collection`) que guían al cliente por la paginación.
- La diferencia práctica: en el servicio Esri el cliente arma las páginas con `resultOffset`; en OGC API sigue los enlaces que le entrega el propio servidor.

## Conclusión

WMS sirve para **ver** (imagen), WFS y los servicios REST sirven para **usar los datos** (geometría y atributos). Los REST (FeatureServer y OGC API) se consumen con una simple petición HTTP y JSON, lo que los hace mucho más cómodos para una página web con Leaflet que el XML + parámetros `KVP` del WMS/WFS clásico.

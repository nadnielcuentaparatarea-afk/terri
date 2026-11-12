# TERRAPULSE — El pulso de la Tierra

Aplicación React + TypeScript + Vite preparada para Netlify. El proyecto parte del ZIP anterior de TerraPulse: conserva la identidad visual, el diseño de paneles, la misión, el catálogo de destinos, el globo Cesium, la consulta STAC y la función ASF; amplía la parte científica con procesamiento SCL real y controles de calidad.

## Funciones incluidas

- Globo 3D de Cesium con cielo estrellado original, atmósfera y cámara animada desde el Caribe hasta el punto de referencia de los Humedales del Ozama.
- Navegación: **La misión**, **Mapa** y **Análisis real**.
- Catálogo de 12 destinos, navegación libre, selección por clic, coordenadas `latitud, longitud`, orientación norte, 2D/3D y vista global.
- Identificación cercana mediante Photon / OpenStreetMap: espera entre clics, cancelación de peticiones antiguas, timeout, caché de sesión y filtro de distancia calculada a 1 km.
- Capas de contexto Esri World Imagery, Esri World Imagery (Clarity) y OpenStreetMap; etiquetas y límites administrativos; atribuciones visibles. Clarity es un servicio legado sin actualizaciones desde 2021 y programado para retiro en marzo de 2028; se limita al máximo nivel 17 o al máximo menor publicado por el servicio.
- Enlace externo a Google Maps en modo satélite. Google no se incrusta como mosaico ni requiere credenciales ni cargos.
- Consulta Sentinel-2 L2A en Earth Search STAC: colección `sentinel-2-c1-l2a`, hasta 60 resultados, filtro de nubosidad de tesela ≤60%, metadatos y selección de dos adquisiciones.
- Procesamiento de los COG SCL con GeoTIFF.js mediante lectura remota por bloques/rangos; no se descargan intencionalmente los GeoTIFF completos.
- Cuadrícula común WGS84 / UTM, resolución de 20 m, muestreo de categorías por vecino más cercano, máscara SCL, margen de exclusión de un píxel, verificación de proyección/alineación/cobertura y umbrales de publicación.
- Métricas reales calculadas únicamente con píxeles válidos comunes: superficie total, área observable, agua anterior/posterior, persistente, nueva, desaparecida y porcentaje de cobertura común.
- Comparador anterior/posterior de SCL, leyenda, gráfica de agua calculada y capa de cambios superpuesta en el globo.
- Exportación GeoJSON con geometría WGS84, cuadrícula/proyección/resolución, adquisiciones, enlaces, métricas, versión de algoritmo, método y limitaciones.
- Consulta del catálogo ASF SearchAPI para `dataset=NISAR` y `processingLevel=GCOV`, hasta 12 productos, metadatos, vista previa cuando ASF la publique y enlace al HDF5 original.
- Laboratorio aparte rotulado permanentemente **SIMULACIÓN EDUCATIVA**, con realces visuales sintéticos, comparador, indicadores, gráfica mensual y línea de tiempo.
- Configuración Netlify SPA, función serverless NISAR, diseño adaptable y soporte de movimiento reducido.

## Requisitos

- Node.js 22 o superior.
- npm 10 o superior.

## Ejecutar en tu computadora

```bash
npm install
npm run dev
```

Vite abrirá normalmente `http://localhost:5173`.

La consulta NISAR usa una Netlify Function. Para probar esa función en local, inicia el entorno de Netlify:

```bash
npx netlify dev
```

## Compilar y probar

```bash
npm test
npm run build
npm run preview
```

La aplicación compilada queda en `dist/`. El proceso de build copia los recursos estáticos de Cesium mediante `vite-plugin-static-copy`.

## Publicar en Netlify

**Recomendado: conectar GitHub.**

1. Descomprime `TerraPulse-Netlify.zip`.
2. Sube el contenido de la carpeta `TerraPulse-Netlify` a un repositorio de GitHub.
3. En Netlify, selecciona **Add new site → Import an existing project** y conecta ese repositorio.
4. `netlify.toml` ya define:
   - Build command: `npm run build`
   - Publish directory: `dist`
   - Functions directory: `netlify/functions`
   - Node: `22`
5. Pulsa **Deploy**.

También puedes desplegar con Netlify CLI:

```bash
npm install
npm run build
npx netlify deploy --prod
```

No se necesita una clave de Cesium Ion para esta configuración: el globo utiliza el elipsoide WGS84 y servicios de imágenes externos. No se incluyen claves privadas ni contraseñas.

## Fuentes y servicios

- Earth Search STAC: <https://earth-search.aws.element84.com/v1>
- Colección Sentinel-2: <https://earth-search.aws.element84.com/v1/collections/sentinel-2-c1-l2a>
- ASF SearchAPI: <https://api.daac.asf.alaska.edu/services/search/param>
- Documentación de parámetros ASF: <https://docs.asf.alaska.edu/api/keywords/>
- Photon: <https://photon.komoot.io/>
- Esri, calendario de retiro de mapas base heredados (Clarity, marzo de 2028; World Boundaries and Places, diciembre de 2029): <https://www.esri.com/arcgis-blog/products/arcgis-living-atlas/announcements/sunsetting-legacy-basemaps>
- OpenStreetMap: <https://www.openstreetmap.org/copyright>
- Esri World Imagery: <https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer> (el proveedor Cesium lee los niveles publicados por el servicio y muestra sus atribuciones).
- Esri World Imagery (Clarity), servicio legado: <https://clarity.maptiles.arcgis.com/arcgis/rest/services/World_Imagery/MapServer>
- Ley 202-04, fuente oficial del Ministerio de Medio Ambiente: <https://ambiente.gob.do/portal-transparencia/app/uploads/wpfd/preview_files/Ley-No.-202-04-Areas-Protegidas-y-de-las-Areas-Especiales%28f73b76ce8949fe29bf2a537cfa420e8f%29.pdf>

## Punto de referencia de los Humedales del Ozama

La Ley 202-04 describe el vértice UTM zona 19N **416350 E / 2058550 N**. Su transformación a WGS84 es aproximadamente **18.616232, -69.792955**. La coordenada aparece como un vértice de la descripción legal de límites; la aplicación la usa solo como punto de referencia y no reconstruye ni inventa el polígono oficial. El área cuadrada que el usuario seleccione tampoco equivale al límite legal.

## Método de cambio de agua

1. Se buscan escenas de `sentinel-2-c1-l2a` y se conservan las que tienen `eo:cloud_cover` ≤60% y un activo SCL disponible.
2. Se seleccionan dos IDs distintos y se ordenan por fecha. El sistema sugiere un par con nubosidad baja y separación temporal.
3. GeoTIFF.js abre los COG SCL mediante solicitudes parciales. Se lee solo la ventana necesaria para el área de estudio.
4. Se verifica que el área esté contenida en ambas teselas, que las dos escenas compartan proyección UTM, resolución SCL de 20 m y alineación de píxeles.
5. El cuadrado se ajusta como máximo 10 m por eje para alinear el origen con la cuadrícula UTM de 20 m. El procesamiento UTM está limitado a latitudes 80° S–84° N; el globo sí permite explorar todo el planeta. Las clases se remuestrean por vecino más cercano; no se interpolan categorías.
6. Agua = clase SCL 6. Dominio admisible = clases 4, 5 y 6. Cualquier otra clase se excluye y se aplica un margen de un píxel alrededor de zonas inválidas.
7. Solo se comparan píxeles válidos en ambas fechas. Se exige al menos 70% de cobertura común y 100 píxeles válidos. Si falla una condición, no se muestran métricas.

SCL es una clasificación de escena, no un mapa validado de uso del suelo. Puede omitir agua bajo vegetación. Agua nueva no confirma una inundación: también puede responder a estacionalidad, mareas o errores de clasificación. No se calculan profundidad de agua, daños, pronósticos ni alertas oficiales.

## NISAR

La función `netlify/functions/nisar.mjs` consulta el catálogo ASF con `dataset=NISAR` y `processingLevel=GCOV`. La aplicación distingue el registro del catálogo, la vista previa oficial (si existe) y el archivo HDF5. **El procesamiento de HDF5 no está implementado**: requiere una cuenta NASA Earthdata y un servicio de procesamiento adecuado. No introduzcas contraseñas en TerraPulse.

## Pruebas y verificación

`npm test` incluye pruebas de:

- Conversión UTM 19N ↔ WGS84 y validación de coordenadas.
- Cuadrado UTM de 20 m.
- Máscara SCL 4/5/6 y margen de un píxel.
- Agua persistente, nueva y desaparecida.
- Rechazo de cobertura insuficiente y menos de 100 píxeles.
- IDs distintos y orden temporal.

La colección STAC y el esquema de activos SCL se comprobaron contra el endpoint público el 12 de noviembre de 2026. La ejecución completa de los dos COG del caso de referencia no pudo completarse en el entorno de construcción, que no tiene salida de red directa para descargar los bloques COG. Por eso **no se afirma que el resultado anterior de 78.1525% se haya reproducido** ni se fijan métricas. Usa el botón **Cargar caso de referencia** después de desplegar o desde `npx netlify dev`; el informe GeoJSON se genera solo si la comparación pasa los controles. Consulta `reports/reference-case.md` para el registro reproducible.

## Atribuciones y limitaciones operativas

- Earth Search, ASF, Photon, Esri y OpenStreetMap son servicios externos; su disponibilidad, cobertura, políticas y latencia pueden cambiar.
- World Imagery y Clarity son mosaicos de contexto, no observaciones históricas de Sentinel-2.
- Clarity está programado para retiro en marzo de 2028 y puede dejar de responder antes por cambios del servicio. La aplicación intenta verificar el servicio y muestra un aviso si falla.
- NISAR: catálogo consultable; procesamiento HDF5 pendiente.
- La función NISAR necesita el runtime de Netlify (Netlify Dev o despliegue en Netlify); Vite por sí solo no ejecuta funciones serverless.

# Establecimientos de Reclusión y Policía en Colombia

Descarga y limpieza de la ubicación de cárceles y sitios de policía en Colombia, pensada para cruzar por distancia con otras bases (por ejemplo, transacciones).

## Fuentes

- **INPEC:** [Directorio de Establecimientos de Reclusión de Orden Nacional](https://www.datos.gov.co/Justicia-y-Derecho/Directorio-de-Establecimientos-de-Reclusi-n-de-Ord/24zf-4cfu/about_data), datos.gov.co, vía la API Socrata (`https://www.datos.gov.co/resource/24zf-4cfu.json`).
- **OpenStreetMap:** cárceles (`amenity=prison`) y sitios de policía (`amenity=police`) vía la [API Overpass](https://overpass-api.de/api/interpreter). Cubre cárceles municipales, departamentales y otras que no dependen del INPEC. La cobertura es desigual fuera de las ciudades.

## Archivos

| Archivo | Contenido |
|---|---|
| `EstablecimientosDeReclusion.ipynb` | Notebook que descarga, limpia y une las fuentes. |
| `establecimientos_reclusion.csv` | 126 establecimientos del INPEC, coordenadas en grados decimales. |
| `carceles_inpec_otras.csv` | INPEC + otras cárceles de OpenStreetMap (sin policía). |
| `lugares_reclusion_policia.csv` | INPEC + otras cárceles + sitios de policía. |

Las tablas unificadas tienen las columnas `tipo` (`INPEC`, `Otra cárcel`, `Policía`), `fuente`, `id_fuente`, `nombre`, `departamento`, `ciudad`, `latitud`, `longitud`.

El notebook incluye además `distancia_a_carceles()`, que recibe una tabla de transacciones con latitud y longitud y agrega la distancia (km) a la cárcel del INPEC y a la otra cárcel más cercanas, con marcas `cerca_inpec` y `cerca_otra_carcel` según un radio. Trae un ejemplo con transacciones **sintéticas** y una celda comentada para cargar la base real desde Excel (`.xlsx`).

Una cárcel de OpenStreetMap se clasifica como INPEC si está a menos de 1 km de un establecimiento oficial (`RADIO_INPEC_KM`); las demás quedan como `Otra cárcel` y conviene revisarlas manualmente, porque algunas coordenadas del INPEC parecen desplazadas.

## Uso

```bash
pip install pandas numpy requests openpyxl
```

Abrir el notebook y ejecutar todas las celdas.

Los datos reales de transacciones no se suben a este repositorio: el `.gitignore` excluye archivos Excel y `transacciones*`. Antes de hacer commit del notebook, borrar los resultados (*Clear All Outputs*) si se corrió con datos reales.

## Licencias

- Datos del INPEC: publicados como datos abiertos en datos.gov.co.
- Datos de OpenStreetMap: © colaboradores de OpenStreetMap, bajo licencia [ODbL](https://www.openstreetmap.org/copyright). `carceles_inpec_otras.csv` y `lugares_reclusion_policia.csv` contienen datos derivados de OpenStreetMap y se distribuyen bajo la misma licencia.

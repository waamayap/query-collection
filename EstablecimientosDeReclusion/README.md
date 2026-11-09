# Establecimientos de Reclusión del Orden Nacional

Descarga y limpieza del [Directorio de Establecimientos de Reclusión de Orden Nacional](https://www.datos.gov.co/Justicia-y-Derecho/Directorio-de-Establecimientos-de-Reclusi-n-de-Ord/24zf-4cfu/about_data) (INPEC), publicado en datos.gov.co.

## Contenido

- `EstablecimientosDeReclusion.ipynb`: consulta la API Socrata (`https://www.datos.gov.co/resource/24zf-4cfu.json`), renombra columnas y convierte las coordenadas de grados-minutos-segundos a grados decimales.
- `establecimientos_reclusion.csv`: resultado limpio (126 establecimientos).

## Columnas del CSV

`regional`, `codigo_divipola`, `departamento`, `ciudad`, `direccion`, `latitud`, `longitud` (decimales), `cod_establecimiento`, `nombre_establecimiento`, `clasificacion_art_11_ley_65`.

## Uso

```bash
pip install pandas requests
```

Abrir el notebook y ejecutar todas las celdas.

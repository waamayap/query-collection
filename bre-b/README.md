# Bre-B Sentinel

Propuesta de detección de fraude para pagos inmediatos Bre-B en Cotrafa, inspirada en el proyecto abierto [PIX Sentinel](https://github.com/guivital1/pix-sentinel). Los datos del repositorio son **sintéticos**.

## Archivos

| Archivo | Contenido |
|---|---|
| `bre-b-sentinel.html` | Página de la propuesta: contexto, monitor de riesgo, ruta del dinero, arquitectura y hoja de ruta. Se abre en el navegador, sin servidor. |
| `RutaDineroBreB.ipynb` | Análisis por pasos: carga el extracto (`.xls`, `.xlsx` o `.csv`), califica cada cuenta y sigue el dinero hacia atrás y hacia adelante. |
| `GenerarEjemploBreB.ipynb` | Crea el archivo de ejemplo con transferencias sintéticas y cuatro tipologías de fraude insertadas. |
| `ejemplo_movimientos_breb.xls` | Archivo de ejemplo (Excel 97-2003): hoja `movimientos` con 6.047 transferencias y hoja `cuentas` con la fecha de apertura. |

## Uso con un archivo propio

```bash
pip install pandas numpy xlrd openpyxl
```

`xlrd` lee `.xls`; `openpyxl` lee `.xlsx`.

1. Copie el archivo en esta carpeta.
2. En la sección 1 de `RutaDineroBreB.ipynb` cambie `ARCHIVO`, `HOJA_MOVIMIENTOS` y `HOJA_CUENTAS`.
3. En `COLUMNAS` escriba cómo se llaman las columnas en su archivo (no importan mayúsculas):

| Nombre en el notebook | Columna en el ejemplo | Contenido |
|---|---|---|
| `tx_id` | `ID_TRANSACCION` | identificador de la transferencia |
| `fecha` | `FECHA` | fecha (`04/11/2026` o `2026-11-04`), con o sin hora |
| `hora` | `HORA` | hora (`10:23:00` o fracción de día de Excel); `None` si va en la fecha |
| `origen`, `destino` | `CUENTA_ORIGEN`, `CUENTA_DESTINO` | cuenta o llave, idealmente seudonimizada |
| `entidad_origen`, `entidad_destino` | `ENTIDAD_ORIGEN`, `ENTIDAD_DESTINO` | entidad de cada lado; Cotrafa se reconoce por el texto `NOMBRE_COTRAFA` |
| `monto` | `VALOR` | valor en pesos (`1.234.567,89`, `$ 1,234,567.89` o número) |

La hoja de cuentas (`CUENTA`, `FECHA_APERTURA`) es opcional: sin ella no se aplica la regla de cuenta nueva.

4. Ejecute todas las celdas. Los resultados quedan en `salidas/`: `cuentas_riesgo.csv`, `rutas_alertas.csv` y `ruta_dinero.json` (los casos del diagrama del HTML).

Los datos reales no se suben: el `.gitignore` de esta carpeta excluye Excel, CSV y `salidas/`, salvo el archivo de ejemplo. Si corrió el notebook con datos reales, borre los resultados (*Clear All Outputs*) antes de hacer commit.

## Qué hace el análisis

PIX Sentinel califica cada transferencia por separado: guarda quién envía y quién recibe, pero no lo usa. Este notebook mira la red de cuentas:

- **De dónde viene:** entradas de la cuenta alertada y, si el origen también tiene riesgo, de dónde le llegó a ese origen.
- **Para dónde va y a dónde sigue:** salidas en las 48 horas siguientes, con el dinero asignado en orden de llegada (primero en entrar, primero en salir), salto por salto.
- **Dónde se detiene:** en una cuenta de Cotrafa sin riesgo (el dinero se mezcla con el saldo del asociado) o cuando sale hacia otra entidad. Cotrafa solo ve una transferencia si una de las dos cuentas es suya.

Reglas por cuenta (tope 100), configurables en la sección 1.3:

| Regla | Puntos |
|---|---|
| Cuenta puente: entra y sale en menos de 1 hora (salidas / entradas entre 0,8 y 1,05) | 40 |
| Ciclo: el dinero vuelve a la cuenta en 48 h | 30 |
| Muchos a uno: 8 o más orígenes en 24 h | 25 |
| Uno a muchos: 8 o más destinos en 24 h | 25 |
| Cuenta con menos de 60 días | 20 |
| Salida de $1 millón o más entre 00:00 y 05:00 | 15 |

Con el archivo de ejemplo se detectan 6 de las 7 cuentas mula con puntaje ≥ 60 y ningún falso positivo en ese nivel; la séptima (`CTF-M06`) queda en riesgo medio y aparece dentro de la ruta del ciclo. Los pesos y umbrales son una propuesta inicial para calibrar con Riesgo y SARLAFT.

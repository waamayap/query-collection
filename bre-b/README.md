# Bre-B Sentinel

Propuesta de detección de fraude para pagos inmediatos Bre-B en Cotrafa, inspirada en el proyecto abierto [PIX Sentinel](https://github.com/guivital1/pix-sentinel). Todos los datos son **sintéticos**.

## Archivos

| Archivo | Contenido |
|---|---|
| `bre-b-sentinel.html` | Página de la propuesta: contexto, monitor de riesgo por transferencia, ruta del dinero, arquitectura y hoja de ruta. Se abre en el navegador, sin servidor. |
| `RutaDineroBreB.ipynb` | Notebook que arma la red de cuentas, califica cada cuenta y sigue el dinero hacia atrás y hacia adelante. |
| `breb_movimientos_sinteticos.csv` | 6.052 transferencias sintéticas de una semana, con `visible_cotrafa` y la tipología insertada. |
| `breb_cuentas_riesgo.csv` | Indicadores de flujo, reglas activadas y puntaje por cuenta de Cotrafa. |
| `ruta_dinero.json` | Casos que muestra el diagrama de la página (también van incluidos dentro del HTML). |

## Ruta del dinero

PIX Sentinel califica cada transferencia por separado: guarda quién envía y quién recibe, pero no lo usa. Este notebook agrega la mirada de red:

- **De dónde viene:** entradas de la cuenta alertada y, si el origen también tiene riesgo, de dónde le llegó a ese origen.
- **Para dónde va y a dónde sigue:** salidas en las 48 horas siguientes, con el dinero asignado en orden de llegada (primero en entrar, primero en salir), salto por salto.
- **Dónde se detiene:** en una cuenta de Cotrafa sin riesgo (el dinero se mezcla con el saldo del asociado) o cuando sale hacia otra entidad. Cotrafa solo ve una transferencia si una de las dos cuentas es suya.

Reglas por cuenta (tope 100):

| Regla | Puntos |
|---|---|
| Cuenta puente: entra y sale en menos de 1 hora (salidas / entradas entre 0,8 y 1,05) | 40 |
| Ciclo: el dinero vuelve a la cuenta en 48 h | 30 |
| Muchos a uno: 8 o más orígenes en 24 h | 25 |
| Uno a muchos: 8 o más destinos en 24 h | 25 |
| Cuenta con menos de 60 días | 20 |
| Salida de $1 millón o más entre 00:00 y 05:00 | 15 |

El generador inserta cuatro tipologías: estafa con estratificación, muchos a uno, uno a muchos y ciclo. Con la semilla 2025 se detectan 6 de las 7 cuentas mula con puntaje ≥ 60 y ningún falso positivo en ese nivel; la séptima (`CTF-M06`) queda en riesgo medio y aparece dentro de la ruta del ciclo. Los pesos y umbrales son una propuesta inicial para calibrar con Riesgo y SARLAFT.

## Uso

```bash
pip install pandas numpy
```

Abrir el notebook y ejecutar todas las celdas. Las salidas se escriben en la misma carpeta.

El notebook trae una celda comentada para cargar datos reales. Úsela solo en el entorno de Cotrafa: los datos reales y sus salidas no se suben a este repositorio (el `.gitignore` de esta carpeta excluye Excel y archivos `movimientos_breb*`/`cuentas_breb*`). Si se corrió con datos reales, borrar los resultados (*Clear All Outputs*) antes de hacer commit.

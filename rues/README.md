# Consulta masiva de NIT en RUES

Consulta una lista de NIT en https://www.rues.org.co/ y extrae la información general, las actividades económicas (CIIU) y los representantes legales (principales y suplentes).

## Estructura

```
Rues/
├── consulta_rues.ipynb     ← notebook principal
├── requirements.txt        ← librerías necesarias
├── entrada/
│   └── nits.xlsx           ← lista de NIT (columna "NIT")
└── salida/
    ├── resultado_rues.xlsx ← resultado (se crea al correr)
    └── progreso/           ← avance y capturas de error (se crea al correr)
```

## Uso

1. Requisitos: Python 3.10 o superior y Google Chrome instalado.
2. Instale las librerías (o ejecute la primera celda del notebook):
   `pip install -r requirements.txt`
3. Ponga los NIT en `entrada/nits.xlsx`, en la columna `NIT`. Sirven con o sin dígito de verificación:
   `901235729`, `9012357298`, `901235729-8` o `901.235.729-8`.
4. Abra `consulta_rues.ipynb` y ejecute las celdas en orden.
   - La sección 8 prueba un solo NIT. Úsela antes de la corrida completa.
   - La sección 10 hace la corrida masiva.

## Resultado (`salida/resultado_rues.xlsx`)

| Hoja | Contenido |
|---|---|
| InfoGeneral | Una fila por NIT: datos generales, CIIU (`ciiu_1`, `ciiu_2`…), los primeros representantes en columnas (`rep1_…`, `rep2_…`) y el texto completo de representación legal |
| Actividades | Una fila por cada código CIIU |
| Representantes | Una fila por persona: tipo (PRINCIPAL/SUPLENTE), cargo, nombre, tipo de documento e identificación |
| Errores | NIT sin resultados en RUES o con error de consulta |

## Notas

- Chrome se abre en una ventana visible porque RUES no carga en modo oculto (headless). No cierre esa ventana mientras corre.
- Si la corrida se interrumpe, vuelva a ejecutar la sección 10: continúa donde iba y reintenta los NIT con `ERROR`.
- Cada NIT tarda unos 15–25 segundos.
- La pestaña de representante legal es texto libre del certificado y cada cámara de comercio lo escribe distinto. El notebook reconoce los formatos de Bogotá, Medellín, Barranquilla y el de lista. Si en algún caso no reconoce a los representantes, el texto completo queda en la columna `texto_representantes`.
- Para empezar de cero, borre la carpeta `salida/progreso`.

# Pronóstico del saldo de financiaciones con tarjeta de crédito en Argentina

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/PJUAREZZ/pronostico-saldo-tarjetas-credito-ar/blob/main/proyecto_integrador.ipynb)

**Proyecto Final Integrador** – Diplomado en Ciencia de Datos y Análisis Avanzado (Centro de e-Learning UTN BA)
**Autor:** Juárez, Pablo Nicolás

## Resumen

Pronóstico a 1, 2 y 3 meses del **saldo de financiaciones con tarjeta de crédito del sistema financiero argentino**, usando la serie pública del BCRA (idVariable 115) como referencia (proxy), ya que los datos de una tarjeta puntual son confidenciales. Está pensado como benchmark para las áreas de Riesgo, Tesorería/Fondeo y Planeamiento Comercial.

- **Variable objetivo:** logaritmo del saldo mensual **nominal**. Las métricas (MAPE, MAE, RMSE) se reportan en pesos nominales, revirtiendo el logaritmo.
- **Comparación:** 4 reglas simples (persistencia, naive estacional, deriva 6 y 12 meses) contra 3 modelos (SARIMA, Ridge, XGBoost).
- **Validación:** walk-forward con 117 fechas de pronóstico entre 2017 y 2026, usando solo información disponible en cada fecha.

## Resultados clave (corrida de octubre de 2026)

MAPE (%) en las últimas 24 fechas de pronóstico:

| Modelo | 1 mes | 2 meses | 3 meses |
|---|---|---|---|
| Persistencia | 3,57 | 6,72 | 10,23 |
| Deriva 6 meses | 2,18 | 3,83 | 5,84 |
| **SARIMA (modelo recomendado)** | 2,18 | 3,58 | 5,39 |
| Ridge | 2,25 | 3,80 | 5,62 |
| XGBoost | 2,11 | 3,47 | 5,16 |

Pronóstico SARIMA (billones de pesos; 1 billón = un millón de millones):

| Mes | Pronóstico | Banda de error 80% | Var. vs. septiembre |
|---|---|---|---|
| Octubre 2026 | 26,06 | 25,09 a 26,87 | +1,2% |
| Noviembre 2026 | 26,49 | 24,94 a 28,12 | +2,9% |
| Diciembre 2026 | 26,98 | 24,73 a 29,73 | +4,8% |

**Conclusiones:**
- Se cumplen los KPI de la pre-entrega: MAPE menor a 12% y mejora de entre 39% y 58% sobre repetir el último saldo.
- No se alcanza la meta más exigente (20% de mejora sobre la mejor regla simple): la prueba de Diebold-Mariano no encuentra diferencias significativas entre SARIMA y la deriva a 6 meses.
- El saldo tiene mucha inercia; la inflación, el tipo de cambio y la tasa aportan poco.

> **Nota de reproducibilidad:** las cifras anteriores corresponden a la ejecución local de octubre de 2026. Al reejecutar el notebook en Colab, las métricas de XGBoost pueden diferir levemente (menos de 0,2 puntos porcentuales de MAPE) por la versión de la librería y la plataforma; el pronóstico central de SARIMA coincide y las conclusiones no cambian.

El detalle completo está en [`docs/Informe_Final_Juarez_Pablo_Nicolas.pdf`](docs/Informe_Final_Juarez_Pablo_Nicolas.pdf).

## Estructura del repositorio

```
.
├── README.md
├── LICENSE
├── requirements.txt
├── data_log.md                 # registro de decisiones del proyecto
├── proyecto_integrador.ipynb   # notebook único: descarga, EDA, variables, baselines, modelos, pronóstico
├── data/                       # series del BCRA (crudas y mensuales) y base de modelado
├── results/                    # tablas de métricas, predicciones, pronóstico y gráficos
└── docs/                       # pre-entrega e informe final (PDF)
```

## Cómo ejecutarlo

### Opción A: Google Colab
1. Hacer clic en el botón **Open in Colab** de arriba.
2. Ejecutar todas las celdas (*Entorno de ejecución → Ejecutar todo*). Tarda unos minutos.

### Opción B: local
```bash
git clone https://github.com/PJUAREZZ/pronostico-saldo-tarjetas-credito-ar.git
cd pronostico-saldo-tarjetas-credito-ar
python3 -m venv .venv
source .venv/bin/activate          # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook proyecto_integrador.ipynb
```
Se necesita acceso a internet para descargar las series de la API del BCRA. Los archivos se generan en `data/` y `results/`.

**Reproducibilidad:** la descarga usa una fecha de corte fija (`FECHA_CORTE = "2026-10-06"`) para obtener los mismos resultados del informe. Para actualizar la serie con datos más recientes, cambiar esa fecha en la primera celda; las cifras variarán.

**Nota:** si XGBoost no puede cargarse (en Mac suele faltar OpenMP: `brew install libomp`), el notebook usa automáticamente `GradientBoostingRegressor` de scikit-learn y lo indica.

## Datos y fuentes

Todas las series provienen de la API pública del BCRA, **Estadísticas Monetarias v4.0** (acceso gratuito y sin registro):

| Variable | idVariable | Frecuencia |
|---|---|---|
| Préstamos mediante tarjeta de crédito otorgados al sector privado (objetivo, millones de ARS) | 115 | Diaria |
| Tipo de cambio mayorista de referencia | 5 | Diaria |
| Tasa BADLAR de bancos privados | 7 | Diaria |
| Inflación mensual (IPC, republicado por el BCRA a partir del INDEC) | 27 | Mensual |

La tasa de política monetaria (ids 160 y 161) se descartó porque dejó de actualizarse el 10/07/2025; se la reemplazó por la BADLAR.

## Metodología (CRISP-DM)

1. **Comprensión del negocio:** objetivo medible, KPI e hipótesis.
2. **Comprensión de los datos:** calidad, valores atípicos (16 detectados y conservados) y EDA.
3. **Preparación:** serie mensual, logaritmo, variables con rezago y control de fuga de información.
4. **Modelado:** reglas simples y modelos, un modelo directo por horizonte.
5. **Evaluación:** walk-forward, MAPE/MAE/RMSE en pesos nominales y prueba de Diebold-Mariano.
6. **Despliegue (propuesta):** actualización mensual y monitoreo del error.

**Control de fuga de información:** el IPC entra con rezago de 1 y 2 meses por su demora de publicación; el tipo de cambio y la BADLAR con su valor al cierre del mes de origen; no se usan valores futuros de ninguna variable; el entrenamiento en cada fecha incluye solo observaciones cuyo resultado ya era conocido.

## Limitaciones

- Serie agregada del sistema: no distingue entidad, segmento ni rubro.
- Cambios de régimen por la inflación; solo 24 fechas de evaluación en el régimen actual.
- Saldo nominal: no descuenta inflación.
- Prophet, mencionado en la pre-entrega, no se incluyó.

## Ética y privacidad

Los datos son públicos y agregados para todo el sistema financiero; no contienen información personal. El modelo **no debe usarse para decisiones de crédito sobre personas**. Se informan siempre las bandas de error y la pérdida de precisión a 3 meses.

## Licencia

Código bajo licencia [MIT](LICENSE). Los datos pertenecen al BCRA y se citan como fuente.

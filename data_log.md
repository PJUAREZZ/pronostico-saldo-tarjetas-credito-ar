# Registro de decisiones (data log)

Registro cronológico de las decisiones técnicas y metodológicas del proyecto.

## Pre-entrega (28/09/2026)

| Decisión | Detalle | Motivo |
|---|---|---|
| Tema | Pronóstico del saldo de financiaciones con tarjeta de crédito (no "pronóstico de ventas") | Los datos de una tarjeta puntual son confidenciales; se usa el saldo del sistema como proxy público y verificable |
| Fuente | API BCRA Estadísticas Monetarias v4.0, idVariable 115 | Acceso gratuito, sin registro y con historia desde 2002 |
| Serie mensual | Último valor de cada mes | Evita el ciclo intramensual (pagos de resúmenes) |
| Variable objetivo | Logaritmo del saldo nominal; métricas en pesos nominales | El saldo crece de forma exponencial; el saldo real exige deflactar con un IPC que se publica con demora |
| Control de fuga | IPC con rezago mínimo de 1 mes; sin valores futuros de variables macro; modelo directo por horizonte | Observación del tutor: el IPC tiene demora de publicación |

## Entrega final (octubre de 2026)

| Decisión | Detalle | Motivo |
|---|---|---|
| Actualización de datos | De 5.808 a 5.826 registros diarios (hasta 06/10/2026); fecha de corte fija `FECHA_CORTE` | Reproducibilidad del informe |
| Mes incompleto | Octubre de 2026 excluido de la serie mensual | Último dato disponible: 06/10/2026 |
| Valores atípicos | 16 detectados (|z| > 4); se conservan sin modificar | Los 9 recientes caen entre los días 7 y 13 de cada mes (probable pago de resúmenes, hipótesis no confirmada) y no entran en la serie mensual; los 7 de 2003-2005 corresponden a una base muy pequeña |
| Tasa de interés | La tasa de política monetaria (ids 160/161) se reemplaza por la BADLAR de bancos privados (id 7) | Las series 160 y 161 dejaron de actualizarse el 10/07/2025 |
| Inflación | Serie 27 del BCRA (republica el IPC del INDEC) | Evita descargar de otra fuente; último dato publicado: agosto de 2026 |
| Variables | 13 variables calculadas solo con información disponible en la fecha t (crecimiento del saldo, IPC rezagado, tipo de cambio, BADLAR, calendario) | Control de fuga de información |
| Baselines | Persistencia, naive estacional, deriva 6 y 12 meses | Se agregan las derivas porque la persistencia es un baseline débil con crecimiento nominal |
| Modelos | SARIMA, Ridge, XGBoost | Mínimo de dos algoritmos exigido por la consigna |
| Prophet | No incluido | Con tres modelos se cumple el mínimo; se priorizó un entorno reproducible |
| Validación | Walk-forward, ventana expansiva, 117 fechas desde enero de 2017; ventanas: desde 2017, desde 2023 y últimas 24 fechas | El contexto cambió mucho; un solo promedio engaña |
| Métricas | MAPE (principal), MAE y RMSE en pesos nominales; prueba de Diebold-Mariano | Métricas de clasificación (F1, AUC) no aplican a regresión |
| KPI | Se mantienen MAPE < 12% y mejora ≥ 20% sobre persistencia; se agrega mejora ≥ 20% sobre la mejor regla simple | La persistencia resultó un baseline débil |
| Modelo elegido | SARIMA como principal; XGBoost como contraste | Error casi igual al de XGBoost, igual o mejor desde 2023, más simple y explicable |
| Banda de error | Cuantiles 10% y 90% de los errores de los últimos 36 meses | No depende de supuestos de normalidad |
| XGBoost en Mac | Requiere OpenMP (`brew install libomp`); si no está disponible se usa `GradientBoostingRegressor` | Compatibilidad del entorno |

## Hallazgos que cambiaron el plan

- La **estacionalidad de diciembre** planteada en la pre-entrega se confirma solo en parte: octubre y noviembre crecen más y febrero casi no crece.
- Las **variables macro** aportan poco por encima de la inercia del crecimiento reciente.
- Ningún modelo logró el **20% de mejora sobre la mejor regla simple**; las diferencias no son estadísticamente significativas.

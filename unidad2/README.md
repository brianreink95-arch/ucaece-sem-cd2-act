# Unidad 2 — Auditoría de equidad en selección laboral

**Estudiante:** Brian Reinke · Licenciatura en Ciencia de Datos · Seminario de Ciencia de Datos II.

Se estudia el sesgo de género en el sistema experimental de reclutamiento de Amazon documentado por Dastin (2018). La práctica utiliza Campus Recruitment para evaluar un mecanismo comparable de inequidad; no reproduce los datos ni el modelo de Amazon.

## Resultados de la entrega

Corrida registrada el 19 de octubre de 2026 en [evidencias.md](evidencias.md).

| Escenario | Exactitud | Demographic parity difference | Equalized odds difference |
|---|---:|---:|---:|
| Modelo base | 0.800 | 0.097 | 0.212 |
| ExponentiatedGradient | 0.800 | 0.038 | 0.101 |

La mitigación redujo ambas disparidades sin pérdida de exactitud global en esta corrida. La tasa de selección del modelo base fue 0.714 en F y 0.811 en M. Persisten diferencias y la muestra es pequeña; las etiquetas reflejan inserción laboral observada, no una evaluación independiente de competencia. Los gráficos SHAP explican únicamente el modelo base. Sus nombres de archivo no prueban discriminación ni su ausencia.

## Dataset y preparación

- **Fuente:** [Campus Recruitment, publicado por Ben Roshan en Kaggle](https://www.kaggle.com/datasets/benroshan/factors-affecting-campus-placement).
- **Archivo original:** `Placement_Data_Full_Class.csv`, 215 filas y 15 columnas. SHA-256 del archivo utilizado: `f4f6ad9e13313a7edd12a71d88adfcec3e5153bf45469199c44e6d47c5de4bd3`.
- **Objetivo:** `status`; clase positiva `Placed` (1), frente a `Not Placed` (0).
- **Atributo sensible:** `gender`, con 76 registros F y 139 M. Se conserva tanto para la auditoría como entre los predictores, de acuerdo con la plantilla.
- **Preparación:** eliminar `sl_no` (identificador) y `salary` (información posterior al resultado). Se preservan las 215 filas y quedan 13 columnas, sin valores faltantes.
- **Modelo:** regresión logística; separación 70/30 estratificada por objetivo y semilla 42; preprocesamiento ajustado sobre entrenamiento.

Los CSV y `.env` no están versionados. Para reproducir la práctica, descargar el archivo original de la fuente y colocarlo en `unidad2/data/`. No aplicar `dropna()` sobre el CSV original: los salarios faltantes corresponden a las personas sin colocación laboral.

## Reproducir la práctica

1. En este fork, abrir **Code → Codespaces → … → New with options**, seleccionar **unidad2-seminario-cd2** y crear el Codespace. Las dependencias se instalan automáticamente desde [requirements.txt](requirements.txt).
2. Verificar que la terminal esté en `unidad2/` y que el CSV original se encuentre en `data/`.
3. Generar el CSV limpio:

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv("data/Placement_Data_Full_Class.csv")
df.drop(columns=["sl_no", "salary"]).to_csv(
    "data/placement_limpio.csv", index=False
)
PY
```

4. Si `.env` todavía no existe, copiar `.env.example` a `.env`. Configurar:

```dotenv
DATASET_PATH=data/placement_limpio.csv
TARGET_COLUMN=status
SENSITIVE_ATTRIBUTE=gender
POSITIVE_LABEL=Placed
MITIGATION_METHOD=exponentiated_gradient
FAIRNESS_CONSTRAINT=demographic_parity
```

5. Ejecutar desde `unidad2/`:

```bash
python -m src.main
```

El flujo genera `evidencias.md` y cuatro PNG en [outputs/](outputs/): comparación de métricas, dos explicaciones individuales y resumen SHAP. La plantilla rota el archivo de evidencias previo a `evidencias.anterior.md`; al repetir la ejecución se deben actualizar las interpretaciones y conservar los gráficos de la misma corrida.

La semilla 42 fija la partición y el modelo base. Las predicciones del reductor pueden variar porque la plantilla llama a `ExponentiatedGradient.predict` sin fijar su estado aleatorio; los valores de la tabla corresponden a la corrida conservada, no garantizan identidad numérica de todas las repeticiones.

## Referencias y alcance

- Dastin, J. (2018, 10 de octubre). [Amazon scraps secret AI recruiting tool that showed bias against women](https://www.investing.com/news/stock-market-news/amazon-scraps-secret-ai-recruiting-tool-that-showed-bias-against-women-1637988). Reuters.
- [Consigna oficial de la cátedra](docs/actividad-unidad2.md).
- [Código de entrenamiento](src/model_training.py) y [selección de explicaciones SHAP](src/explainability.py).

El dataset requiere aprobación docente según la consigna. Esta documentación no atribuye una aprobación que no se haya registrado.

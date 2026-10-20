# Comparativas algorítmicas

Estos notebooks respaldan las decisiones del pipeline en `../Proyecto_EDA_Motor_Ecommerce_REES46_v2.ipynb`. Se ejecutan igual que ese notebook: abrir el archivo y correr todas las celdas.

Cada notebook trae su propio código. No importa archivos `.py`.

## Cómo se ejecuta

| Parámetro | Valor inicial | Efecto |
|---|---|---|
| `RUN_MODE` | `"quick"` | Mide 1K, 10K y 100K |
| `RUN_MODE` | `"full"` | Añade 1M y 10M. En esos tamaños baja a 3 repeticiones y la tabla lo registra |
| `SEED` | `42` | Semilla reproducible |
| `DATASET_FILE` | `""` | Vacío: muestra sintética, solo para validar el notebook |
| `DATASET_FILE` | `2019-Oct.csv` | Evidencia real. Puede estar en esta carpeta o en la raíz del proyecto |

Para el informe y la exposición usa `full` y el CSV de octubre. Si la columna `fuente` dice `SINTÉTICO`, ese resultado no se cita.

| Notebook | Decisión que sostiene |
|---|---|
| `00_todas_las_comparativas.ipynb` | Regenera las cinco evidencias en una corrida |
| `01_filtrado_probabilistico.ipynb` | Bloom y Count-Min frente al estado exacto |
| `02_similitud_conjuntos.ipynb` | MinHash+LSH frente a SimHash y Jaccard |
| `03_vecinos_aproximados.ipynb` | HNSW frente a IVF y KNN exacto |
| `04_grafo.ipynb` | PageRank y Louvain frente a grado y Label Propagation |
| `05_estructuras_base.ipynb` | Hash y KD-Tree frente a las búsquedas exactas |

Las tablas quedan en `resultados/` y las figuras en `figuras/`. El notebook también las muestra al ejecutarse. Instala `hnswlib` antes de la corrida final de vecinos: sin ese paquete el experimento avisa y omite HNSW.

```bash
python -m pip install numpy pandas matplotlib networkx hnswlib notebook
jupyter notebook pruebas_algoritmos/00_todas_las_comparativas.ipynb
```

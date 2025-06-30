# Index String Format

The list of index strong formats for `faiss.index_factory(...)`.

---

## Flat Indexes (no training)

| `index_format`    | Description                             |
| -------------- | --------------------------------------- |
| `IDMap,Flat`   | Exact brute-force search                |
| `IDMap,HNSW`   | HNSW graph index with default neighbors |
| `IDMap,HNSW32` | HNSW with 32 neighbors                  |
| `IDMap,HNSW64` | HNSW with 64 neighbors (more accurate)  |

---

## IVF (Inverted File) Indexes – Require Training

| `index_format`          | Description                    |
| -------------------- | ------------------------------ |
| `IDMap,IVF64,Flat`   | 64 clusters, no quantization   |
| `IDMap,IVF256,Flat`  | 256 clusters, fast/compact     |
| `IDMap,IVF1024,Flat` | 1024 clusters                  |
| `IDMap,IVF4096,Flat` | 4096 clusters, higher accuracy |

---

## IVF + Scalar Quantization (SQ) – Require Training

| `index_format`            | Description                               |
| ---------------------- | ----------------------------------------- |
| `IDMap,IVF256,SQ8`     | 256 clusters + 8-bit scalar quantization  |
| `IDMap,IVF1024,SQ8`    | 1024 clusters + SQ8                       |
| `IDMap,IVF1024,SQfp16` | 1024 clusters + 16-bit float quantization |

---

## IVF + Product Quantization (PQ) – Require Training

| `index_format`          | Description                    |
| -------------------- | ------------------------------ |
| `IDMap,IVF256,PQ8`   | IVF + PQ with 8 subquantizers  |
| `IDMap,IVF1024,PQ16` | IVF + PQ with 16 subquantizers |
| `IDMap,IVF4096,PQ32` | IVF + PQ with 32 subquantizers |
| `IDMap,PQ16`         | Standalone PQ without IVF      |

---

## Composite Indexes

| `index_format`                    | Description                 |
| ------------------------------ | --------------------------- |
| `IDMap,PCA64,IVF1024,Flat`     | Reduce dim to 64 then index |
| `IDMap,OPQ64_128,IVF1024,PQ64` | Optimized PQ pipeline       |

---

## Binary Indexes


| `index_format`     | Description                         |
| --------------- | ----------------------------------- |
| `IDMap,BFlat`   | Binary flat index with external IDs |
| `IDMap,BIVF256` | Binary IVF index with 256 clusters  |
| `IDMap,BHNSW`   | Binary HNSW                         |




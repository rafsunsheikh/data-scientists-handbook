# Graph / Network Data

> **TL;DR** — Graph data represents *entities* (nodes) and *relationships* (edges). Anything where the relationship matters as much as the attributes — social networks, supply chains, fraud rings, citation graphs, knowledge bases — is fundamentally graph data. The biggest practical decision is whether to materialize the graph in memory or query it from a graph database.

## 1. Sub-types

| Sub-type | Description | Example |
|---|---|---|
| Undirected | Edges have no direction | friendship |
| Directed (digraph) | Edges point one way | follower, citation |
| Weighted | Edges carry a numeric weight | distance, transaction amount |
| Multigraph | Multiple edges between same pair | repeated transactions |
| Bipartite | Two node types, edges only across | user ↔ product |
| k-partite / heterogeneous | Many node and edge types | knowledge graph |
| Temporal / dynamic | Edges have timestamps or appear/disappear | message logs |
| Hypergraph | Edges connect >2 nodes | co-authorship of >2 |
| Property graph | Nodes and edges carry attributes | most real-world graphs |
| Tree / DAG | Special case: hierarchical or acyclic | org chart, dependency graph |

## 2. Storage representations

| Representation | When |
|---|---|
| Edge list (`(u, v)` or `(u, v, w)`) | Compact, easy to stream |
| Adjacency list | Iterating neighbors |
| Adjacency matrix | Small dense graphs; spectral analysis |
| CSR / CSC sparse matrix | Large sparse graphs; numerical libs |
| Property graph store (Neo4j, TigerGraph, Memgraph) | Production transactional queries |
| RDF triple store (Jena, Blazegraph) | Knowledge graphs with reasoning |
| Pandas DataFrames (nodes.csv + edges.csv) | Analysis-friendly |
| Parquet / DuckDB | Large analytical workloads |

## 3. Properties to characterize

| Property | How |
|---|---|
| Size | `|V|`, `|E|` |
| Density | `2E / (V(V-1))` for undirected |
| Degree distribution | histogram of node degrees |
| Connected components | `nx.connected_components` |
| Diameter / average path | BFS sampling for large graphs |
| Clustering coefficient | local triangle density |
| Centralities | degree, betweenness, closeness, eigenvector, PageRank |
| Community structure | Louvain, Leiden, label propagation |
| Assortativity | do similar nodes connect to similar nodes? |

Most real-world graphs are **scale-free** (power-law degree distribution) and **small-world** (low diameter, high clustering). Models that assume Erdős-Rényi random graphs will be wildly wrong on them.

## 4. Common pitfalls

1. **Self-loops and multi-edges** not handled by the chosen algorithm.
2. **Disconnected graphs.** Many algorithms (shortest path, eigenvector centrality) misbehave or return per-component results.
3. **Node ID inconsistency.** Strings vs. ints vs. mixed across sources. Canonicalize first.
4. **Edge direction reversed.** "Alice follows Bob" vs. "Bob is followed by Alice" — pick one and stick to it.
5. **Memory.** Adjacency matrix for 1M nodes is 1 TB. Use sparse representation.
6. **Sampling bias in network data.** A snowball-sampled friendship network underrepresents isolated nodes.
7. **Train/test leakage** in graph ML. Random edge split leaks structure — use inductive or temporal splits.
8. **PII via graph topology.** Even anonymized graphs can be re-identified by neighborhood structure [^backstrom].

## 5. Cleaning checklist

- [ ] Canonicalize node IDs.
- [ ] Decide policy for self-loops and parallel edges.
- [ ] Remove or flag isolated nodes.
- [ ] Identify and inspect large connected components.
- [ ] Deduplicate edges.
- [ ] Validate edge attributes (e.g., weights ≥ 0 if required).

## 6. Algorithms to know

| Family | Algorithms | Use |
|---|---|---|
| Traversal | BFS, DFS | Reachability, shortest unweighted |
| Shortest path | Dijkstra, Bellman-Ford, A* | Routing |
| Centrality | Degree, betweenness, closeness, PageRank | Influence |
| Community | Louvain, Leiden, label propagation | Cluster detection |
| Matching | Bipartite matching, stable marriage | Assignment |
| Flow | Max-flow / min-cut | Capacity problems |
| Embedding | node2vec, DeepWalk, GraphSAGE, GNNs | ML on graphs |
| Link prediction | Common neighbors, Adamic-Adar, GNN | Friend / product recs |
| Anomaly detection | Egonet analysis, OddBall | Fraud rings |

## 7. Visualizations

See [`data_visualization/network_viz.md`](../data_visualization/network_viz.md).

| Question | Chart |
|---|---|
| Overall topology (small graph) | Force-directed layout |
| Degree distribution | Histogram on log-log axes |
| Adjacency / connectivity pattern | Adjacency matrix heatmap |
| Communities | Color nodes by community in force layout |
| Centrality | Size nodes by centrality |
| Temporal evolution | Animation, or small multiples per timestep |

For graphs above ~10k nodes a full force layout becomes hairball. Use sampling, edge bundling, matrix views, or aggregation to "supernode" level.

## 8. References

- Newman. *Networks*, 2nd ed.
- Easley & Kleinberg. *Networks, Crowds, and Markets* (free online).
- Hamilton. *Graph Representation Learning Book* (free online).
- NetworkX, PyG (PyTorch Geometric), DGL documentation.

[^backstrom]: Backstrom, L., Dwork, C., & Kleinberg, J. (2007). *Wherefore Art Thou R3579X? Anonymized Social Networks, Hidden Patterns, and Structural Steganography.*

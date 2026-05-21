# Network Visualization

> **TL;DR** — Force-directed layouts are useful for ~10 to ~1,000 nodes. Above that, every layout becomes a hairball. For large graphs, switch to adjacency matrix views, edge bundling, hierarchical layouts, or supernode aggregation.

## Pick a viz

| Graph size | Useful viz |
|---|---|
| ≤ ~30 nodes | Hand layout; circular; force-directed |
| 30–1,000 nodes | Force-directed with community coloring |
| 1,000–100,000 nodes | Edge bundling; adjacency matrix; node aggregation |
| > 100,000 nodes | Sample or aggregate first; matrix views; visual exploration tools (Gephi, Cytoscape, neo4j-bloom) |

## Force-directed layouts

```python
import networkx as nx
import matplotlib.pyplot as plt

pos = nx.spring_layout(G, seed=0, k=0.5)
nx.draw_networkx(G, pos,
    node_size=[v * 50 for v in dict(G.degree).values()],
    node_color=[community[n] for n in G.nodes],
    edge_color='lightgray', with_labels=False, alpha=0.8)
```

Tips:

- Set the random seed — layouts are non-deterministic.
- Size nodes by degree or centrality.
- Color nodes by community.
- Use very faint edges if there are many; the structure comes through.

Algorithms beyond `spring`: ForceAtlas2 (`fa2`), OpenOrd (good for clusters), Yifan Hu (for very large graphs).

## Circular layout

Place all nodes on a circle, draw chords inside. Useful when:

- Nodes have a natural order (chromosomes in a chord diagram of genome data).
- You want symmetry to highlight density rather than community.

`nx.circular_layout`, `nx.chord_diagram`, `circlize` (R), `holoviews.Chord`.

## Hierarchical layout

For trees and DAGs:

```python
pos = nx.nx_agraph.graphviz_layout(G, prog='dot')   # requires pygraphviz
```

Reveals depth and parent-child structure.

## Adjacency matrix view

When the graph is dense or large, a matrix is more readable than a layout:

```python
import seaborn as sns
A = nx.to_pandas_adjacency(G)
order = community_order(G)              # reorder rows/cols by community
sns.heatmap(A.loc[order, order], square=True, cbar=False)
```

Communities show up as block-diagonal patterns.

## Arc diagram

Nodes on a line; edges drawn as arcs above. Excellent for showing which nodes are most-connected without the hairball.

## Sankey / alluvial (for flow graphs)

If your "graph" is really a flow between layers (user funnel, ETL flow), a Sankey diagram is more readable than any node-link layout. See [`compositions.md`](compositions.md).

## Edge bundling

Pull together edges with similar paths to reduce visual clutter.

Tools: `datashader`'s `hammer_bundle`, `pygraphviz` with edge bundling, Gephi.

## Animations and dynamic graphs

For temporal networks:

- **Animated layout** — interpolate between layouts at each time step. Watch for flicker; "anchor" stable nodes.
- **Small multiples** — one snapshot per time step, side by side. Often more useful than animation.
- **Streaming view** — running window of recent edges (e.g., in fraud / cyber monitoring).

## Avoid

1. **The hairball** — force layout on 100k nodes. Add structure first (community detection, sampling).
2. **3-D layouts.** Worse than 2-D for almost all tasks.
3. **Cluttered labels.** Label only the highest-degree or selected nodes.
4. **Rainbow community colors** for many communities. Use a palette designed for categorical data (`tab20`, `glasbey`).
5. **Reporting graph metrics without showing the graph** — and vice versa.

## Tools beyond Python

- **Gephi** — interactive exploration, ForceAtlas2, community detection.
- **Cytoscape** — biology-flavored; very feature-rich.
- **neo4j Bloom** — exploration of Neo4j graphs.
- **graph-tool** (Python) — faster than NetworkX for large graphs.

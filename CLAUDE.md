# CLAUDE.md: Network Science HS26 Assignment (Enzyme Structural Graph)

Context for Claude. Read this whole file before you edit the notebook or draft the report.

## 1. The task

- **Course:** Network Science HS26, UZH (Blockchain & DLT group, C. J. Tessone). Lectures covered so far: L1 degree and distributions, L2 paths, clustering, assortativity and randomisation, L3 centrality, L4 random graphs and small worlds, L5 communities.
- **Assignment:** "Bioinformatics Network Investigation". The student investigates one **undirected, unweighted enzyme structural graph** and gives an evidence-based description of how it is organised.
- **Deliverables:** (1) a written report, (2) the executed notebook, (3) figures and calculations that reproduce every claim.
- The notebook follows the 8 open questions of the brief as sections 1–8. Section 0 (loading and overview figure) is already done.
- The brief warns: *"The goal is not to calculate as many network measures as possible."* A few well-chosen measures with interpretation beat a wall of numbers. When in doubt, leave a measure out.

## 2. Repo layout and running

```
NetworkScience/                      # repo root (git)
├── CLAUDE.md
├── README.md                        # layout and setup; keep in sync if the layout changes
├── NetworkSciene_HS26_Assignment.pdf  # the brief
├── requirements.txt                 # networkx, numpy, pandas, scipy, matplotlib, ipykernel
├── raw_data/edgelist.txt            # supplied data, never modify
├── code/assignmentNotbook.ipynb     # analysis notebook; uses ../raw_data and ../figures
├── figures/                         # every saved PNG (network_overview.png exists)
├── report/                          # written report goes here
├── tmp/                             # scratch, not part of the submission
└── venv/                            # local Python env (git-ignored); never edit
```

- The submitted notebook must be **executed, with outputs visible**. After editing, run from the repo root:
  `source venv/bin/activate && jupyter nbconvert --to notebook --execute --inplace code/assignmentNotbook.ipynb`
  `nbconvert` is not in `requirements.txt` or the venv yet. **Ask the user before installing it** (then also add it to `requirements.txt`). Otherwise the user runs "Run All" in the editor and saves.
- The setup cell defines `DATA`, `FIG`, `SEED = 42`. The overview cell defines `nodes`, `deg`, `pos` (Kamada–Kawai), `deg_color`, `node_size` and `A`.
  **Reuse them and never recompute the layout.** Every node-link figure uses the same `pos`, so a reader can find a node across figures.
- Leave the overview cell (§0) as it is unless the user asks for a change.
- The venv has networkx 3.7. All snippets here were tested with networkx 3.6.1 and 3.7.
- Don't commit or push unless the user asks.

## 3. Grading checklist (from the brief): every section must serve it

1. Use the supplied network. **Compute every number in the notebook.** Never hard-code values, including the sanity-check values in §10 of this file.
2. Separate **observation, measurement and interpretation**, using the template in §7.
3. Make every interesting node or feature **locatable**: give node IDs (edges as `u–v`) and highlight them in a figure on the shared layout.
4. Never describe the network with a single measure. Back each conclusion with two or more pieces of evidence.
5. Every comparison with a model states: which model, why it was chosen, what it keeps fixed, what was compared, and how many samples and which seeds were used.
6. Back conclusions with **evidence from this network**, not with textbook descriptions of concepts.
7. Make **≥ 3 claims that depend on analysing this network**, and have **≥ 1 of them name specific nodes, edges or groups**. Collect them in a final "Key claims" cell (§8 below).
8. The notebook must be reproducible from the edge list, and every reported number must trace back to a code output.

## 4. Data facts and gotchas

- The file has **220 rows: 110 unique edges, each listed in both directions.** This contradicts the brief, which says directions are not listed twice. `nx.Graph` merges the duplicates. Report it as a data check (the notebook already does).
- 51 nodes (IDs 1–51), no self-loops, no isolated nodes, one connected component.
- The brief says IDs don't imply importance. But the **ID order is not random**: 33 of the 50 pairs (i, i+1) are linked, while a degree-preserving null gives about 4–5. Treat this as a finding (§8), not an assumption.
- **Domain background (a hypothesis, not stated in the brief).** In standard enzyme or protein structure graph datasets (e.g. ENZYMES or PROTEINS in TUDataset), nodes are secondary-structure elements (helices, strands). Edges join elements that are neighbours along the amino-acid sequence or close in 3D space. That would explain:
  - the consecutive-ID backbone,
  - the narrow degree range, since physical packing limits the number of contacts,
  - the long, lattice-like distances.

  Use this only to motivate interpretations, phrased as "consistent with …". Never present it as a fact.
- The graph is undirected and unweighted, so reciprocity, strength, HITS and the bow-tie structure don't apply, and PageRank adds little beyond degree. Leave them out.

## 5. Code style: keep it simple (the user's explicit wish)

A grader should understand each cell in about 30 seconds.

- **Libraries:** `networkx`, `numpy`, `pandas`, `matplotlib`, `scipy.stats`. Nothing else (no seaborn, powerlaw, igraph, python-louvain or sklearn) unless the user asks.
- **One cell, one idea**, about 25 lines at most. Follow each result with a markdown interpretation cell.
- Use plain loops, dicts and DataFrames. No classes, no nested comprehensions, no clever one-liners.
- **At most three helper functions in the whole notebook:** `metrics(H)`, `id_runs(nodes)` and an optional `draw_values(ax, values, title)` for coloured node-link plots.
- Comments explain **why**, not what Python does. For example: `# keeps every node's degree, shuffles who is linked to whom`.
- Show results as small pandas tables with `.round(3)`. No long print dumps.
- Pass `seed=SEED` to everything random, or `seed=i` inside sample loops. Never change seeds to get nicer numbers.
- **Figures:**
  - Use the shared `pos` and the blue degree palette from the overview. Use `plt.cm.tab10` for categories such as communities.
  - Give each figure one message and axis labels that say what is shown.
  - Save every figure as `FIG / "<section>_<name>.png"` with `dpi=150, bbox_inches="tight"`.
- Degrees only span 2–6, so plot distributions with **linear axes and integer bins** (L1: log bins only for heavy-tailed data). Plot p(k), not raw counts.
- Don't compute anything that the text doesn't interpret.

## 6. Course definitions and the matching networkx calls

| Concept (lecture) | networkx | Note |
|---|---|---|
| ⟨k⟩, density ρ = 2L/(N(N−1)) (L1) | `2 * L / N`, `nx.density(G)` | |
| distance, ⟨d⟩, diameter, eccentricity, periphery, centre (L2) | `nx.average_shortest_path_length`, `nx.diameter`, `nx.eccentricity`, `nx.periphery`, `nx.center` | |
| clustering Cᵢ, network ⟨C⟩ (L2, L4) | `nx.clustering`, `nx.average_clustering` | In this course ⟨C⟩ is the **mean of Cᵢ**. Use `nx.transitivity` only as a side check. |
| k_nn,i and k_nn(k) (L2) | `nx.average_neighbor_degree`, then group by degree | |
| assortativity r (L2) | `nx.degree_assortativity_coefficient` | Pearson correlation of the degrees at both ends of each link |
| degree, closeness, betweenness, eigenvector (L3) | raw degree, `nx.closeness_centrality`, `nx.betweenness_centrality`, `nx.eigenvector_centrality(G, max_iter=1000)` | Closeness = (N−1)/Σd, as in L3. Betweenness is normalised by default (say so). Eigenvector has unit norm, which gives the same ranking as the course's sum normalisation. |
| rank correlation (workbook §3) | `cent.corr(method="spearman")` and `"kendall"` | |
| degree-preserving randomisation (L2, L4) | `nx.random_reference(G, niter=10, connectivity=False, seed=i)` | Returns a new graph. `connectivity=False` is the workbook hint and about 100× faster. All samples come out connected anyway, and `metrics()` uses the largest component to be safe. |
| Erdős–Rényi G(N, L) (L4) | `nx.gnm_random_graph(N, L, seed=i)` | Keeps N and L exactly. Can be disconnected (about 40 % of samples), so ⟨d⟩ is computed on the largest component. Say so. |
| ring lattice and Watts–Strogatz (L4) | `nx.watts_strogatz_graph(N, 4, p, seed=...)` | p = 0 is the ring lattice: κ = 2 gives k = 4 ≈ ⟨k⟩. Assortativity is undefined for a regular graph (NaN), so drop that entry. |
| communities (L5, workbook §5) | `nx_comm.greedy_modularity_communities` (CNM, deterministic), `nx_comm.louvain_communities(G, seed=SEED)`, `nx_comm.modularity` | label propagation is optional. The leading eigenvector method is not in networkx, so skip it. |
| strong and weak communities (L5) | compare k_int and k_ext per node and per community | |
| robustness basics | `nx.bridges`, `nx.articulation_points`, `nx.node_connectivity` | |

p-value wording (L4): the p-value is the share of null samples at least as extreme as the observed value. It is **not** the probability that H0 is true. With 200 samples, an observed share of 0 means p < 1/200.

## 7. Markdown template after every result

> **Measurement:** what was computed and how (function, null model, number of samples, seed), plus the number.
> **Observation:** the pattern visible in the table or figure, with node IDs.
> **Interpretation:** what it means for *this* enzyme graph, hedged ("consistent with …").
> **Caveat:** the main limitation (small N, dependence on seed or algorithm, choice of model).

Write 1–3 sentences per line and no generic textbook text.

## 8. Section plan: what to compute, show and answer

### Shared reference-model cell (add as §1.2; reused in §4–§8)

```python
from networkx.algorithms import community as nx_comm

N, L = G.number_of_nodes(), G.number_of_edges()
N_SAMPLES = 200

def metrics(H):
    """Global numbers we compare against the reference models in §4–§8."""
    lcc = H.subgraph(max(nx.connected_components(H), key=len))  # <d> is only defined on a connected graph
    return {
        "avg_clustering": nx.average_clustering(H),
        "avg_path_length": nx.average_shortest_path_length(lcc),
        "assortativity": nx.degree_assortativity_coefficient(H),
        "modularity": nx_comm.modularity(H, nx_comm.louvain_communities(H, seed=SEED)),
        "consecutive_id_edges": sum(abs(u - v) == 1 for u, v in H.edges()),
    }

# Degree-preserving: every node keeps its degree, links are shuffled -> "is it more than the degrees?"
refs_dp = [nx.random_reference(G, niter=10, connectivity=False, seed=i) for i in range(N_SAMPLES)]
# Erdős–Rényi G(N, L): only size and number of links are kept -> "is it more than the density?"
refs_er = [nx.gnm_random_graph(N, L, seed=i) for i in range(N_SAMPLES)]

observed = pd.Series(metrics(G))
null_dp = pd.DataFrame([metrics(R) for R in refs_dp])
null_er = pd.DataFrame([metrics(R) for R in refs_er])
```

This takes about 10 seconds. §4–§6 cite single rows of these tables, and §7 shows the whole comparison.

### §1 Connectivity and global structure
- **Compute:** one small table with N, L, ρ, ⟨k⟩, number of components, diameter, ⟨d⟩, radius, centre, periphery, bridges, articulation points and node connectivity.
- **Figure:** nodes coloured by eccentricity on the shared layout, with the periphery and the centre labelled.
- **Answer:**
  - The network is one sparse component (ρ ≈ 0.09).
  - It is **2-connected**: no bridge and no articulation point, so no single node or link failure splits it.
  - It is **elongated**: diameter 12 against ⟨d⟩ ≈ 5.
  - It has two far-apart "ends", {1, 2, 17, 34, 42} and {15, 16, 30, 31}, with node 25 as the unique centre (eccentricity 6).
  - Point ahead to §7: ⟨d⟩ is far above random (≈ 2.8). The shape is consistent with a folded chain.

### §2 Important or unusual nodes
- **Compute:**

  ```python
  cent = pd.DataFrame({
      "degree": deg,
      "closeness": nx.closeness_centrality(G),
      "betweenness": nx.betweenness_centrality(G),
      "eigenvector": nx.eigenvector_centrality(G, max_iter=1000),
  })
  ```

  Then show the top 5 per measure, `cent.corr(method="spearman")` and the top edges by `nx.edge_betweenness_centrality`.
- **Figure:** a 2×2 panel on the shared layout, with node colour showing each centrality.
- **Answer:**
  - **Degree barely separates nodes.** Nine nodes tie at the maximum k = 6, and the full range is only 2–6.
  - **Betweenness reveals a corridor.** The top nodes are 8, 25, 26, 36 and 27. The top edges, 36–8, 8–25, 25–26 and 26–27, each carry about 34–39 % of shortest paths: one chain linking the left half to the right module. Of these nodes only 25 has maximum degree; 26, 27 and 36 have k = 4.
  - **Eigenvector centrality is localised on the dense left region** (43, 35, 33, 47, 20, 5). Nodes 39 and 49 have k = 6 but an eigenvector score of about 0.004, against 0.31 for 43 and 35. In an elongated graph this measure picks out "embedded in the densest patch", not global importance.
  - **The measures disagree.** Spearman correlations are about 0.26 for betweenness–eigenvector and 0.37 for degree–betweenness. L3 says rankings agree in nested networks; this one is not nested.
  - **Unusual nodes:**
    - 36: k = 4, C = 0, high betweenness and neighbours in three communities, so a pure connector.
    - 11: k = 2, yet one of only two entry points into the right module.
    - 25: the network's centre.

### §3 Degree structure
- **Compute:** p(k), mean, standard deviation, min and max, and variance/mean. Overlay `stats.poisson.pmf(k, ⟨k⟩)` (what ER predicts) and show the distribution of k_max over `refs_er`.
- **Figure:** a bar chart of p(k) with Poisson points, linear axes, k = 0…10.
- **Answer:**
  - ⟨k⟩ = 4.3 hides that the distribution is **narrow and under-dispersed**: 45 % of nodes have k = 4, all nodes lie in 2–6, and variance/mean ≈ 0.27 (a Poisson distribution gives 1).
  - ER graphs with the same N and L give hubs up to k ≈ 9 (never k_max ≤ 6 in 200 samples). About 95 % of them contain a node with k ≤ 1. The real graph has neither.
  - So the graph is **more regular than random**, the opposite of scale-free. This is consistent with physically limited contacts.
  - Caveat: the distribution comes from only 51 values.

### §4 Degree correlations (assortativity)
- **Compute:** r, with its z-score and p from `null_dp`. Compute k_nn(k) for the real graph and its mean over `refs_dp` by looping over the references and grouping by degree.
- **Figure:** observed k_nn(k) as line and markers, the degree-preserving mean as a dashed line, with both r values in the legend (as in workbook §2).
- **Answer:**
  - The graph is **mildly assortative**: r ≈ +0.17, against −0.04 ± 0.10 in the null (p ≈ 0.02–0.03).
  - k_nn(k) rises from 3.5 (k = 2) to 4.7 (k = 6), while the null curve stays flat at about 4.6.
  - High-degree nodes sit together. For example, 33, 35, 43 and 47 all have k = 6 and share 5 of the 6 possible links. The k = 2 nodes (2, 11, 17) attach mainly to k = 3–4 nodes.
  - Newman's table (L2) gives r ≈ −0.16 for protein *interaction* networks. This graph behaves like a spatial contact graph instead.
  - Caveat: there are only 5 distinct degrees and 110 edges, so call the effect "mild".

### §5 Local structure: triangles and clustering
- **Compute:** Cᵢ, ⟨C⟩, transitivity, triangle count, nodes with C = 0, maximal cliques (`nx.find_cliques`), and C compared with `null_dp` and `null_er`.
- **Figure:** nodes coloured by Cᵢ, with the 4-clique highlighted.
- **Answer:**
  - ⟨C⟩ ≈ 0.24, about 3.5× the degree-preserving null (0.06) and the ER null (0.08). The clustering is real and not driven by the degrees.
  - There are 32 triangles, for example 5–6–7 and 20–21–22. The only 4-clique is {37, 41, 45, 51}.
  - Triangles live inside the dense patches: 28 of the 32 lie within one Louvain community (link to §6). The 4 that cross communities (4–5–35, 4–33–35, 19–20–43, 22–23–44) all sit in the ambiguous left half.
  - Nodes with C = 0 include the connectors 36 and 11 and the peripheral nodes 17, 18, 34 and 42. 14 and 29 have no triangles but sit in 4-cycles (grid-like packing).
  - ⟨C⟩ is below a ring lattice (0.5), so the packing is local but not perfectly regular.

### §6 Communities and mesoscopic structure
- **Compute:**
  - the greedy (CNM) and Louvain (`seed=SEED`) partitions, with number of communities and Q,
  - Q against `null_dp` (z-score),
  - Louvain stability over 100 seeds: number of distinct partitions and the range of Q,
  - a per-community table: size, internal edges, edges leaving, nodes with k_int ≤ k_ext,
  - `id_runs` per community (feeds §8).
- **Figure:** nodes coloured by community, internal edges dark and between-community edges light grey (workbook hint), greedy and Louvain side by side on the shared layout.
- **Answer:**
  - Q ≈ 0.59 (greedy, 4 communities) and 0.61 (Louvain, 5 communities). That is far above the degree-preserving null (0.41 ± 0.02, at most 0.47 in 200 samples), so the structure is not explained by the degrees.
  - **How convincing each community is:**
    - **Very convincing:** {11–16, 27–31, 38–40, 48–50}, 16–17 nodes and 33–34 internal edges, attached to the rest by **only 2 edges** (26–27, plus 11–12 or 10–11 depending on where node 11 goes). Greedy finds it, and so do all 100 Louvain runs. Label propagation, which is noisier, splits it.
    - **Stable:** the core {9, 10, 25, 26, 37, 41, 45, 51}, which contains the 4-clique. It is never regrouped across Louvain runs.
    - **Ambiguous:** the left half (roughly nodes 3–8, 17–24, 32–36, 42–47) is split differently by greedy and Louvain and regrouped across seeds: 13 distinct Louvain partitions in 100 runs, Q 0.607–0.625. Several of its nodes have k_int ≤ k_ext (3, 4, 17, 23, 36, 44).
    - **Small group {1, 2, 3, 17}:** its total degree, 12, is below √(2L) ≈ 14.8, which puts it at the **resolution limit** (L5). Treat it with caution.
  - **Caveat:** a ring lattice with the same N and k reaches Q ≈ 0.65. An elongated graph can be cut into chunks with few boundary edges, so part of the high Q reflects geometry. The 2-edge attachment is the strongest evidence. No community is "strong" in the Radicchi sense.

### §7 Comparison with random, small-world and scale-free models
- **Compute:**

  ```python
  comparison = pd.DataFrame({
      "observed": observed,
      "deg-preserving mean": null_dp.mean(), "deg-preserving sd": null_dp.std(),
      "ER mean": null_er.mean(), "ER sd": null_er.std(),
  })
  comparison["z vs deg-preserving"] = (comparison["observed"] - comparison["deg-preserving mean"]) / comparison["deg-preserving sd"]
  comparison["p (share of null >= observed)"] = (null_dp >= observed).mean()
  comparison.round(3)
  ```

  Add a ring-lattice row for ⟨C⟩ and ⟨d⟩ only. An optional Watts–Strogatz sweep over p ∈ {0, 0.01, 0.03, 0.05, 0.1, 0.2, 1} uses 50 samples each and `nx.connected_watts_strogatz_graph`.
- **Answer.** For each model give the evidence for, the evidence against and a verdict:
  - **Random (ER):**
    - For: it matches sparsity and connectivity. ⟨k⟩ = 4.3 > ln N = 3.9 puts ER at the edge of its connected regime (about 60 % connected in simulation).
    - Against: clustering is 3× higher, ⟨d⟩ is 1.8× longer, there are no hubs, the degrees are under-dispersed, and the graph has modules and the ID backbone.
    - Verdict: **not random.**
  - **Small-world:**
    - For: high C relative to random.
    - Against: paths are not short. ⟨d⟩ ≈ 5.0, against 2.8 for random and ln N/ln⟨k⟩ ≈ 2.7, and closer to the ring lattice's 6.8. Watts–Strogatz only reaches ⟨d⟩ ≈ 5 at p ≈ 0.03–0.05, where its C ≈ 0.45, about double ours.
    - Verdict: a **lattice-like "large world" with local clustering**, not small-world.
  - **Scale-free:**
    - Against: k_max = 6, and a degree range of 2–6 cannot span orders of magnitude. The degrees are under-dispersed against Poisson and there are no hubs.
    - **Don't fit a power law**; with 5 distinct degree values the fit is meaningless.
    - Verdict: not scale-free.
  - **Why these models:** ER keeps only the density, the degree-preserving null keeps every degree (the stronger null), and the ring lattice keeps N and k ≈ 4 (the regular, spatial end of Watts–Strogatz). Each null uses 200 samples with seeds 0–199.
  - **Limitations:**
    - N = 51 is small.
    - Small-world is a statement about how ⟨d⟩ scales with N (⟨d⟩ ∝ ln N), which one fixed-size graph cannot test.
    - The null models ignore spatial embedding.
    - The graph has no weights or coordinates.
    - Results depend on algorithm and seed.

### §8 Beyond the summary
**Primary feature: the node-ID backbone.**

```python
def id_runs(nodes):
    """Split node IDs into runs of consecutive integers, e.g. {1, 2, 3, 7} -> [[1, 2, 3], [7]]."""
    runs = []
    for n in sorted(nodes):
        if runs and n == runs[-1][-1] + 1:
            runs[-1].append(n)
        else:
            runs.append([n])
    return runs
```

- **Observation:** the §0 adjacency matrix shows a band along the diagonal plus off-diagonal stripes.
- **Measurement:**
  - 33 of the 50 pairs (i, i+1) are linked, against 4.6 ± 2.0 in the degree-preserving null (z ≈ 14; at most 10 in 200 samples).
  - The 17 missing pairs mark the chain breaks. The longest unbroken runs are 17–28 and 1–7.
- **Combine with §6:** each community is a few contiguous ID segments packed together. For example, the right module consists of 11–16, 27–31, 38–40 and 48–50.
- **Why it matters:** it explains the long ⟨d⟩ and diameter (a chain folded on itself), the modules (segments far apart in the order but in contact) and the corridor (where segments meet). None of this shows in ⟨k⟩, ⟨C⟩, r or the degree distribution. It appears only through the node order, the adjacency matrix and a null test.
- **Optional second feature** (pick one if there is room): the eigenvector localisation from §2, or the corridor 36–8–25–26–27 together with the 2-edge cut to the right module.

### Final cell: "Key claims" (also the backbone of the report)
1. The module {11–16, 27–31, 38–40, 48–50} is attached to the rest of the network only through edges 26–27 and 11–12 (or 10–11).
2. Shortest paths between the two halves funnel through the corridor 36–8–25–26–27 (the top-4 edges by edge betweenness). So betweenness ranks degree-4 nodes at the top, while degree cannot separate the nine k = 6 nodes.
3. The node IDs encode a chain: 33 of 50 consecutive pairs are linked, against about 4.6 expected.
4. Clustering is high (0.24 against 0.06) while paths are long (5.0 against 2.8): the graph is lattice-like, not small-world.
5. The degrees are under-dispersed (variance/mean 0.27) with no hubs, so the graph is not scale-free.

## 9. Report guidance
- Use the same 8 headings and the template from §7. Reference figures by filename. Numbers must match the notebook output, rounded to 2–3 decimals.
- Open with a 4–5 sentence summary, list the key claims, and close with limitations.
- The brief sets no length or format. **Ask the user before drafting**; don't invent page limits.

## 10. Sanity-check values (networkx 3.6.1, checked with 3.7; SEED = 42; nulls use seeds 0–199)

For checking notebook output only. **Never paste these into the notebook.** Null values shift slightly with networkx version.

- **Basics:** N = 51, L = 110 (220 file rows), ρ = 0.086, ⟨k⟩ = 4.314, connected, 0 self-loops.
- **Degrees:** k = 2/3/4/5/6 for 3/6/23/10/9 nodes. SD 1.08, variance/mean 0.27.
  - k = 6: nodes 5, 20, 25, 33, 35, 39, 43, 47, 49.
  - k = 2: nodes 2, 11, 17.
- **Distances:** diameter 12, ⟨d⟩ = 5.042, radius 6, centre {25}, periphery {1, 2, 15, 16, 17, 30, 31, 34, 42}.
- **Robustness:** no bridges, no articulation points, node and edge connectivity 2.
- **Clustering:** ⟨C⟩ = 0.238, transitivity 0.244, 32 triangles, largest clique {37, 41, 45, 51}. C = 0 nodes: 11, 14, 17, 18, 29, 34, 36, 42.
- **Degree correlations:** r = +0.169. k_nn(k) for k = 2…6: 3.50, 4.06, 4.64, 4.66, 4.70. The degree-preserving null is flat at 4.64 → 4.53.
- **Centralities:**
  - Betweenness top: 8 (0.438), 25 (0.432), 26 (0.381), 36 (0.381), 27 (0.366).
  - Edge betweenness top: 36–8 (0.385), 26–27 (0.378), 8–25 (0.370), 25–26 (0.338).
  - Closeness top: 8 (0.276), 25 (0.270), 36 (0.265).
  - Eigenvector top: 43 and 35 (0.307), 33 and 47 (0.301), 20, 5. Nodes 39 and 49 score about 0.004.
  - Spearman: degree–betweenness 0.37, degree–eigenvector 0.42, betweenness–eigenvector 0.26, closeness–betweenness 0.62, closeness–eigenvector 0.61.
- **Communities:**
  - Greedy: 4 communities, Q = 0.593.
  - Louvain (seed 42): 5 communities, Q = 0.613.
  - Louvain over 100 seeds: 13 distinct partitions, Q 0.607–0.625, 4 or 5 communities. The right module appears in all 100 runs.
  - The right module's only external edges are 11–12 or 10–11, and 26–27.
  - 28 of the 32 triangles lie inside one Louvain community.
- **Degree-preserving null:** ⟨C⟩ 0.063 ± 0.025, ⟨d⟩ 2.797 ± 0.034, r −0.035 ± 0.102 (p ≈ 0.025), Q 0.411 ± 0.019 (max 0.465), consecutive-ID edges 4.6 ± 2.0 (max 10).
- **ER G(N, L):** ⟨C⟩ 0.081 ± 0.026, ⟨d⟩ (largest component) 2.775 ± 0.056, Q 0.398 ± 0.019, k_max mean 9.0 (min 7), connected in 60 % of samples, ln N/ln⟨k⟩ = 2.69.
- **Lattice and Watts–Strogatz (N = 51, k = 4):** ring lattice C = 0.50, ⟨d⟩ = 6.76, Q ≈ 0.65. At p = 0.05, C ≈ 0.44 and ⟨d⟩ ≈ 4.65. At p = 1, C ≈ 0.07 and ⟨d⟩ ≈ 2.9.

## 11. Don'ts
- Don't add measures that aren't interpreted: no PageRank, HITS, reciprocity, k-core tables, power-law fits or attack simulations unless the user asks.
- Don't call the network "small-world" or "scale-free".
- Don't present the protein background (§4) as fact.
- Don't edit `raw_data/`, don't change seeds to tune results, and don't leave cells unexecuted or failing.

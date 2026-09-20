# Graph Report - packing checklist 0.1  (2026-09-20)

## Corpus Check
- 265 files · ~364,362 words
- Verdict: corpus is large enough that graph structure adds value.
- Unclassified: 13 file(s) not represented in the graph (top: .log 7, .woff2 5, (none) 1)

## Summary
- 81 nodes · 49 edges · 37 communities (6 shown, 31 thin omitted)
- Extraction: 67% EXTRACTED · 33% INFERRED · 0% AMBIGUOUS · INFERRED: 16 edges (avg confidence: 0.94)
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- Community 0
- Community 1
- Community 2
- Community 3
- Community 4
- Community 5
- Community 6
- Community 7
- Community 8
- Community 9
- Community 10
- Community 11
- Community 12
- Community 13
- Community 14
- Community 15
- Community 16
- Community 17
- Community 18
- Community 19
- Community 20
- Community 21
- Community 22
- Community 23
- Community 24
- Community 25
- Community 26
- Community 27
- Community 28
- Community 29
- Community 30
- Community 31
- Community 32
- Community 33
- Community 34
- Community 35
- Community 36

## God Nodes (most connected - your core abstractions)
1. `scripts` - 5 edges
2. `scripts` - 5 edges
3. `paths` - 4 edges
4. `paths` - 4 edges
5. `media` - 2 edges
6. `media` - 2 edges
7. `$schema` - 1 edges
8. `registry` - 1 edges
9. `blocks` - 1 edges
10. `components` - 1 edges

## Surprising Connections (you probably didn't know these)
- `outbound-checklist-demo` --composition--> `demo snapshots`  [INFERRED]
   →   _Bridges community 4 → community 5_

## Import Cycles
- None detected.

## Hyperedges (group relationships)
- **temporal_sequence** — frame-00-at-32s, frame-01-at-36s, frame-02-at-40s, frame-03-at-66s [0.95]
- **workflow_phases** — composition-04-auto-advance, composition-05-over-target-stop, composition-06-match-report [0.9]

## Communities (37 total, 31 thin omitted)

### Community 0 - "Community 0"
Cohesion: 0.20
Nodes (9): authoringSkill, media, autoProxy, paths, assets, blocks, components, registry (+1 more)

### Community 1 - "Community 1"
Cohesion: 0.22
Nodes (8): media, autoProxy, paths, assets, blocks, components, registry, $schema

### Community 2 - "Community 2"
Cohesion: 0.22
Nodes (8): name, private, scripts, check, dev, publish, render, type

### Community 3 - "Community 3"
Cohesion: 0.22
Nodes (8): name, private, scripts, check, dev, publish, render, type

### Community 4 - "Community 4"
Cohesion: 0.48
Nodes (7): auto-advance composition, over-target-stop composition, match-report composition, demo thumbnails, outbound-checklist-demo, pallet verification, warehouse scanning workflow

### Community 5 - "Community 5"
Cohesion: 0.33
Nodes (6): contact sheet, demo snapshots, frame 00 at 32s, frame 01 at 36s, frame 02 at 40s, frame 03 at 66.445s

## Knowledge Gaps
- **28 isolated node(s):** `$schema`, `registry`, `blocks`, `components`, `assets` (+23 more)
  These have ≤1 connection - possible missing edges or undocumented components. (Counts symbols only; 63 node(s) total have ≤1 connection when file, concept and rationale nodes are included.)
- **31 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Are the 6 inferred relationships involving `demo snapshots` (e.g. with `contact sheet` and `frame 00 at 32s`) actually correct?**
  _`demo snapshots` has 6 INFERRED edges - model-reasoned connections that need verification._
- **What connects `$schema`, `registry`, `blocks` to the rest of the system?**
  _28 weakly-connected nodes found - possible documentation gaps or missing edges._
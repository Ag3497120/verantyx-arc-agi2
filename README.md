# Verantyx ARC-AGI-2

**Pure program synthesis solver for ARC-AGI-2** — no neural networks, no LLMs, no hardcoded patterns.

## Score

| Split | Score | Method |
|---|---|---|
| Training (1000 tasks) | **74/1000 = 7.4%** | DSL synthesis + CEGIS verification |

## Philosophy

Verantyx approaches ARC-AGI-2 through **structural reasoning**:

1. **Decompose** — Break input→output relationships into composable transformations
2. **Synthesize** — Generate candidate programs from a rich DSL of grid operations  
3. **Verify** — CEGIS (CounterExample-Guided Inductive Synthesis) against ALL training pairs
4. **Compose** — Chain 2-step pipelines when single rules don't suffice

**Zero learning from test outputs.** The solver only uses training examples to discover patterns, then applies verified programs to test inputs.

## Architecture

```
Input Grid → Candidate Generation → CEGIS Verification → Test Application
                    ↓                       ↓
              DSL Operations          All Training Pairs
              (50+ rules)            Must Match Exactly
```

### DSL Operations (50+)

**Cell-level rules** (CellRule):
- `colormap`, `mirror_h/v/hv`, `transpose`, `scale`, `gravity`
- `symmetry_fill`, `cond_neighbor`, `fill_enclosed`, `diagonal_tile`
- `mod_copy`, `mod_copy_flip`, `self_tile`, `extract_at`

**Whole-grid transforms** (WholeGridProgram):
- **Geometric**: `rotate_90/180/270`, `flip_h/v`, `reverse_rows/cols`, `roll_rows/cols`
- **Structural**: `crop_bbox`, `crop_to_color`, `extract_largest/smallest_region`, `extract_unique_subgrid`
- **Morphological**: `erode`, `dilate`, `hollow_regions`, `fill_interior`, `extract_border`
- **Color**: `colormap`, `replace_color`, `recolor_by_size`, `keep_one_color`, `remove_color`
- **Sorting**: `row_sort`, `col_sort` (by color count, sum, etc.)
- **Gravity**: `gravity_all/up/down/left/right`
- **Connection**: `connect_h/v/hv`, `spread_color` (4 directions)
- **Tiling**: `tile_to_output`, `corners_mirror`, `stack_h/v/h_flip/v_flip`
- **Subgrid**: `subgrid_select/overlay/diff` (separator detection)
- **Deduplication**: `dedup_rows`, `dedup_cols`
- **Learned**: `neighborhood_rule` (radius 1-2 neighborhood mapping)

**Composition**: 2-step `CompositeProgram` pipelines (e.g., `crop_bbox + tile_to_output`)

### Key Innovation: Neighborhood Rule Learning

The most powerful single operation: learns a deterministic mapping from each cell's local neighborhood (3×3 or 5×5) to its output value. Solves **240/1000** training tasks alone.

## Usage

```bash
# Download ARC-AGI-2 data
git clone https://github.com/arcprize/arc-agi-2.git /tmp/arc-agi-2

# Run evaluation on training split
python -m arc.eval_cross --split training

# Solve a single task
python -c "
from arc.cross_solver import solve_task_cross
result = solve_task_cross('/tmp/arc-agi-2/data/training/00576224.json')
print(result)
"
```

## Score Progression

| Version | Score | Key Change |
|---|---|---|
| v1 | 16/1000 (1.6%) | Initial DSL: colormap, mirror, scale, gravity |
| v5 | 25/1000 (2.5%) | WholeGridProgram class, rotations |
| v10 | 29/1000 (2.9%) | Subgrid ops, CompositeProgram |
| v12 | 41/1000 (4.1%) | extract_region, stack ops |
| v14 | 53/1000 (5.3%) | corners_mirror, connect ops |
| v17 | 61/1000 (6.1%) | neighborhood_rule learning |
| **v19** | **74/1000 (7.4%)** | **+18 DSL ops, priority reorder** |

## Requirements

- Python 3.8+
- No external dependencies (pure Python)

## Related

- [Verantyx V6](https://github.com/Ag3497120/verantyx-v6) — HLE (Humanity's Last Exam) solver using the same structural reasoning philosophy
- [ARC-AGI-2](https://arcprize.org/) — The benchmark

## License

MIT

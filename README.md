# Scaled defect tables

Per-model records of where a full-corpus run failed and why. A defect attribution
typed once by hand is carried forward mechanically while the failure is unchanged.

Set `DEFECTS_PATH` to this directory to use it. The tools do not find it on their
own: with `DEFECTS_PATH` unset they keep the older `Models/*-denylist.txt` path.

## Files

| File | Written by | Holds |
| --- | --- | --- |
| `zerocopy.tsv` | `sj reconcile <run directory>` | one row per model of a full-corpus ZeroCopy verify run |
| `classes.tsv` | `sj reconcile <run directory>` | one row per model of a full-corpus Classes verify run |
| `compile.tsv` | `sj compile-binary-all` | one row per model the publisher skipped |
| `defects.md` | by hand | one block per defect key |

## Format

Tab separated. The first line begins with `#` and names the run directory and the
commit sha of CSharpGenerators, ScaledCompilers, and OmiSpecifications at the time
of that run; a checkout that is absent is written as `unknown`. The second line is
the column header. Rows follow, sorted by model identifier.

Inside a cell, a backslash, tab, carriage return, and newline are written as `\\`,
`\t`, `\r`, and `\n`, so one row is always one line.

`zerocopy.tsv` and `classes.tsv`:

    model  model_hash  stage  error  prev_stage  prev_error  defect  prev_defect  run

`compile.tsv`:

    model  error  prev_error  defect  prev_defect  run

- `stage` is `passed`, `generation`, `build`, or `packets`. A `packets` row is a model that built but failed a pcap fixture.
- `error` is the first error of that stage, with the run directory removed from
  every path and the `(line,col)` position removed from C# diagnostics. Everything
  else is verbatim. It is empty when the stage is `passed`.
- `model_hash` is the sha256 of the `.model.json`. It is recorded, not compared.
- `defect` is a short key into `defects.md`. A failing row with no key yet holds
  `TODO`; a passed row holds an empty cell.
- `run` is the run directory that produced the row.
- A model absent from the run has no row. Git history holds the old row.

## Rules

- Copy-forward: when `stage` and `error` equal the previous row, `defect` is copied
  and the `prev_` columns are carried forward unchanged. Otherwise the previous
  `stage`, `error`, and `defect` move into `prev_stage`, `prev_error`, and
  `prev_defect`, and `defect` becomes `TODO` for a human to fill.
- `verify-*-all` reads its table for expectations and never writes it. A failing
  model whose row has the same stage and error is expected. A different stage or
  error, or a failing model with no row, is a regression and the exit code is
  nonzero. A model that now passes while its row says it failed is reported, not
  failed. The Small and Medium tiers read the same table and ignore rows for models
  outside their folder. With no table file present, every failure is a regression.
- `reconcile <run directory>` merges one full-corpus verify run into its table. It
  refuses any other run.
- `compile-binary-all` writes `compile.tsv` after it replaces the live tree.
- No recipe commits, pulls, or pushes this repository.

After a reconcile, `grep -n $'\tTODO\t' *.tsv` lists every cell to fill. Fill
only those cells, then commit.

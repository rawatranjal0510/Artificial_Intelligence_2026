# Classical Reasoning Agent: KB + Query → True/False

**AI Assignment 2026, IIT Jodhpur**
Pranjal Rawat · Roll No. M24MA2008 · M.Sc. Mathematics and Data and Computational Science

A deterministic system that decides whether a query is entailed by a knowledge base of facts and rules. Rule-based parsers (regex and recursive descent) translate the input into literals, rules, CNF clauses or a causal model, and a router sends it to a classical engine implemented from scratch. **No generative model is used at any stage; every True/False answer comes from the engine.**

| Input type | Engine |
|---|---|
| RuleTaker-style English, prefix logic, ProbLog | Semi-naive forward chaining with negation as failure |
| First-order logic (FOLIO) | Set-of-support resolution refutation with unification |
| Counterfactual questions (CounterBench) | Causal propagation over a structural causal model |

## Files

| File | Contents |
|---|---|
| `reasoning_agent.ipynb` | Complete implementation: parsers, forward chaining, resolution, causal propagation, evaluation and submission writer |
| `report.pdf` | Representation, algorithms, public-set experiments, error analysis and reasoning traces |
| `submission.csv` | Predictions, columns `id,label` (label is `True`/`False`) |
| `README.md` | This file |

## Results on public data

| Dataset | Queries | Accuracy | Time (s) |
|---|---:|---:|---:|
| RuleTaker dev, closed-world (`naf=True`) | 3000 | 100.00% | 1.81 |
| RuleTaker dev, open-world (`naf=False`) | 3000 | 74.93% | 1.69 |
| RuleTaker test | 5000 | 99.92% | 3.65 |
| CounterBench | 500 | 94.60% | 0.12 |
| FOLIO validation | 203 | 84.24% | 25.0 |

Times are total engine time (parsing included) on a Google Colab CPU runtime. For FOLIO, `Uncertain` counts as False; a constant-False baseline scores 64.5%.

## Requirements

- Python 3.8+ (tested on Google Colab)
- `pandas`, `pyarrow`, `datasets`, `huggingface_hub`

On a fresh environment, uncomment the install cell at the top of the notebook, or run:

```bash
pip install datasets huggingface_hub pandas pyarrow
```

The reasoning engines themselves use only the Python standard library.

## Data

All datasets are downloaded from the Hugging Face Hub by the notebook; nothing needs to be downloaded by hand.

| Dataset | Hub repo | Access |
|---|---|---|
| RuleTaker | `hitachi-nlp/ruletaker` | public |
| CounterBench | `CounterBench/CounterBench` | public |
| FOLIO | `yale-nlp/FOLIO` | **gated**: accept the terms on the dataset page, then uncomment and run the `login()` cell and paste a read token |

Do not save a token inside the notebook.

## How to run

1. Open `reasoning_agent.ipynb` (Jupyter or Google Colab) and run all cells top to bottom.
2. Section 8 prints example reasoning traces. Sections 9 and 10 print accuracy, per-category accuracy and total inference time for each dataset.
3. **To produce `submission.csv` for the private test file:** in the last code cell set

   ```python
   PRIVATE_TEST = "path/to/private_test.csv"
   ```

   and run it. `.csv`, `.json`, `.jsonl` and `.parquet` are accepted. Columns are detected automatically:

   | Role | Column names tried, in order |
   |---|---|
   | context | `premises-FOL`, `given_info`, `context`, `theory`, `premises`, `knowledge_base`, `kb`, `story`, `logic` |
   | query | `conclusion-FOL`, `question`, `statement`, `conclusion`, `query`, `hypothesis` |
   | id | `id`, `ID`, `example_id`, `idx` (row index if none) |

   If the names differ, pass them explicitly: `predict_file(PRIVATE_TEST, out="submission.csv", naf=NAF, ctx="...", q="...", idc="...")`.
   With `PRIVATE_TEST = None` the cell writes predictions for the 5,000-query public RuleTaker test sample instead.
4. `NAF = True` (closed-world reading) is the default because it matches the RuleTaker labels.

To query the agent directly after running the code cells:

```python
ok, (engine, trace) = solve(
    "Bob is young. Bob is kind. If someone is young, then they are smart. "
    "If someone is smart and kind, then they are successful.",
    "Bob is successful.",
)
# ok == True, engine == "forward-chaining"
# successful(bob)  [rule 1]
#   smart(bob)  [rule 0]
#     young(bob)  [given]
#   kind(bob)  [given]
```

## Known limitations

- **FOLIO**: unbalanced parentheses are repaired heuristically; premises that still fail to parse, or whose CNF exceeds 20,000 clauses, are skipped (this can only make a proof harder, never create a false one). Equality has only reflexivity and symmetry axioms. Resolution uses a per-query time budget (1 s in the reported run).
- **CounterBench**: weaker on conditional (92.78%) and backdoor (86.02%) questions, because observations of derived variables are not propagated back to their causes.
- **RuleTaker**: 4 of 5,000 test errors, all in depth-3 / depth-3ext stories with negated rule bodies.
- Runtimes depend on the machine; reported times are from a Colab CPU runtime.

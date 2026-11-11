# Classical Logical Reasoning System

AMAN KUMAR | M26MAC001

The system checks whether a query follows from input facts and rules. It uses
forward chaining, backward chaining, resolution and DPLL for inference.

## Directory structure

```text
submission/
    Data/                    Seven public dataset files
    reasoning_agent.ipynb    Complete implementation and saved results
    reasoning_agent.py       Same implementation for PyCharm
    submission.csv           Combined predictions from all seven files
    README.txt
```

## Representation and pipeline

```text
Facts and rules -> Logical formulas -> Clauses -> Reasoning -> Prediction
```

- Facts and rules use predicates, constants and variables.
- Formulas are stored as trees and converted to conjunctive normal form (CNF).
  Each CNF clause is an OR of literals; the clauses are joined by AND.
- Function-free formulas are grounded over their constants.
- CounterBench descriptions are translated into Boolean equations for the
  observed situation and the situation after an intervention. DPLL decides
  whether the resulting equations imply the query.

The parser accepts formal logic and a limited set of English sentence patterns.
FOLIO is evaluated using its supplied `premises-FOL` and `conclusion-FOL` fields.
Malformed formulas are reported as errors. Brackets and symbol names are not
automatically repaired or matched by spelling similarity.

## Reasoning methods

| Method | Operation | Selection |
|---|---|---|
| Forward chaining | Start with facts and apply rules whose premises are known. | Ground Horn clauses with zero or several constraints. |
| Backward chaining | Start with a goal and prove the premises of rules that conclude it. | Ground Horn clauses with one constraint. |
| DPLL | Use unit clauses, pure literals and branching to test satisfiability. | Other Boolean clauses and encoded CounterBench inputs. |
| Resolution | Match complementary literals and derive clauses until a contradiction is found or the search ends. | Clauses with function terms. |

A Horn clause has at most one positive literal. A constraint has no positive
literal. The negated query commonly supplies the single constraint used by
backward chaining. Both chaining methods can also be selected explicitly with
`horn_method="forward_chaining"` or `horn_method="backward_chaining"`.

The entailment test checks whether KB AND NOT query is inconsistent. If so,
the query is entailed. Parsing, grounding, CNF conversion and unification support
the four methods; they do not provide separate prediction algorithms.

## Requirements

Python 3.10 or later. The reasoning code uses the Python standard library.
A notebook editor is needed for the `.ipynb` file. The `.py` file runs directly
in PyCharm or a terminal.

## Run the notebook

1. Extract the complete folder, including `Data`.
2. Open `reasoning_agent.ipynb` and select a Python kernel.
3. Run the cells in order.
4. Section 7 prints checks and reasoning examples for all four methods.
5. Section 8 evaluates the selected files and writes `submission.csv`.

If the kernel starts in another folder, set `BASE_DIR` in the last cell to the
full path of the extracted `submission` folder.

To use PyCharm, open the folder, select a Python interpreter and run
`reasoning_agent.py`. Its paths are relative to the script.

## Run another input file

Place the file in `Data` and change the `dataset_files` list in the last notebook
cell, or near the bottom of the Python script:

```python
dataset_files = ["evaluation.jsonl"]
```

Run the definition cells before the last cell. JSON, JSONL and CSV are accepted.
An input record can contain formal facts and rules:

```json
{"id":"q1","facts":["Bright(iris)"],"rules":["forall x (Bright(x) -> Visible(x))"],"query":"Visible(iris)"}
```

Another accepted form is:

```json
{"id":"q2","premises":["A","A -> B"],"query":"B","input_format":"logic"}
```

The readers also accept RuleTaker `context`/`statement`, FOLIO formal fields,
and CounterBench `given_info`/`question`. Reference answers are not required.
Unlabelled files display accuracy as N/A.

Supported operators are `~`, `&`, `|`, `->`, `<->`, `^`, `forall` and `exists`,
along with their Unicode forms. Use parentheses to state the scope clearly.
Names are case-sensitive. Names beginning `__` are reserved. Explicit quantifiers
bind variable names; unbound x, y, z, u, v, w, X, Y, Z and ?names in term positions
are treated as variables. Other term names are constants.

## Public results

The following values come from a complete notebook run. No fitting step is used;
files named train are evaluated in the same way as other files. Runtime includes
parsing and reasoning, excluding file loading. The limit is one second per query.

| Dataset | Queries | Correct | Accuracy | Unresolved | Time (s) |
|---|---:|---:|---:|---:|---:|
| `ruletaker_train.jsonl` | 1,000 | 983 | 98.30% | 0 | 1.341 |
| `ruletaker_test.jsonl` | 250 | 244 | 97.60% | 0 | 0.364 |
| `ruletaker_ood_test.jsonl` | 375 | 373 | 99.47% | 0 | 0.500 |
| `counterbench_alpha.json` | 1,000 | 905 | 90.50% | 165 | 3.545 |
| `counterbench_backdoor.json` | 200 | 175 | 87.50% | 10 | 0.617 |
| `FOLIO_train.jsonl` | 1,001 | 774 | 77.32% | 234 | 44.547 |
| `FOLIO_validation.jsonl` | 203 | 164 | 80.79% | 62 | 3.738 |
| **Total** | **4,029** | **3,618** | **89.80%** | **471** | **54.652** |

FOLIO True maps to True. False and Uncertain both map to False for binary
entailment. Accuracy includes every labelled record, including unresolved cases
that receive the CSV fallback. Timing and search-limit outcomes can vary with
hardware and load. These are public development results.

## CSV and result details

`submission.csv` contains 4,029 rows from all seven public files, with columns
`id,predicted_label`. Each row is calculated by the reasoning code. No record
is dropped. When several files are combined, IDs use `filename:source_id` to
avoid collisions. For a single file, original IDs are preserved. If an input
has no ID, its zero-based row number is used. Duplicate output IDs are rejected.

True means entailed. False means not entailed; it does not imply that the
opposite statement was proved. An input error or search limit gives
`prediction=None` and a separate status. The required Boolean CSV uses False
for these unresolved cases. They are counted in the Unresolved column above.

After a notebook run, `summary` contains metrics, `details` contains individual
results and traces, and `predictions` contains every combined output record.
Each result records its method, status and runtime. The metrics also include
accuracy on resolved records and the number of fallback labels that matched
reference answers. Labels, IDs and stored explanations are excluded from the
input sent to the solver.

## Checks and limits

The notebook runs 16 small checks and shows four reasoning examples. Development
validation also passed 43 tests and 1,180 generated checks for new names, rule
order, removed facts and misleading labels. The tests include independent truth
tables and checks of both chaining methods on cyclic rules.

The English grammar is limited. CNF conversion and grounding may grow large.
General first-order resolution may reach its search limit. Equality with function
terms is unsupported. Under classical logic, an inconsistent KB entails every
query. CounterBench input assumes acyclic deterministic Boolean equations;
inconsistent observations are reported separately.

## Dataset sources

- [RuleTaker](https://huggingface.co/datasets/jise/ruletaker)
- [FOLIO](https://aclanthology.org/2024.emnlp-main.1229/)
- [CounterBench](https://huggingface.co/datasets/CounterBench/CounterBench)
AI Assignment 2026 - Classical Reasoning Agent (KB + query -> True/False)
Author: Prashant, Roll No. M24MA2009, IIT Jodhpur

FILES
  reasoning_agent.ipynb  complete implementation (parsers, forward chaining, resolution,
                         causal propagation, evaluation, submission writer)
  report.pdf             representation, algorithms, public-set experiments, reasoning traces
  submission.csv         columns: id,label  (label is True/False)
  README.txt             this file

HOW IT WORKS (one paragraph)
  A rule-based front-end (regex / recursive-descent parsers, no generative AI) converts the
  knowledge base and query into literals, rules, CNF clauses or a causal model. A router then
  picks a classical engine: forward chaining (with negation as failure) for RuleTaker-style
  KBs, resolution refutation with unification for first-order logic (FOLIO), and causal
  propagation for counterfactual questions (CounterBench). The True/False answer always
  comes from that engine.

REQUIREMENTS
  Python 3.8+ (tested on Google Colab), pandas, pyarrow, datasets, huggingface_hub.
  First cell installs them if needed:  pip install -q datasets huggingface_hub pandas pyarrow

DATA (all loaded from Hugging Face by the notebook)
  RuleTaker     hitachi-nlp/ruletaker
  CounterBench  CounterBench/CounterBench
  FOLIO         yale-nlp/FOLIO   (gated: accept the terms on the dataset page, then run
                                  huggingface_hub.login() and paste a read token;
                                  do NOT save the token inside the notebook)

HOW TO RUN
  1. Run the notebook top to bottom. Evaluation cells print accuracy and total inference time.
  2. Public-set results from the submitted run: RuleTaker dev 100.00% (naf=True),
     RuleTaker test 99.92% (5000 queries, 3.65 s), CounterBench 94.60% (500, 0.12 s),
     FOLIO validation 84.24% (203, 25.0 s). FOLIO "Uncertain" counts as False.
  3. To produce the final submission.csv on the private test file, set PRIVATE_TEST in the
     cell near the end to the file's path and run it. Columns are auto-detected
     (context: premises-FOL / given_info / context / ...; query: question / statement / ...;
     id: id / example_id / ...). If they differ, pass ctx=, q=, idc= to predict_df(...).
     Without an id column the row index is used as id.
  4. NAF = True (closed-world reading) is the default; it matches the RuleTaker labels.

NOTES / KNOWN LIMITS
  - FOLIO: unbalanced parentheses are repaired heuristically; premises that cannot be parsed,
    or whose CNF exceeds 20,000 clauses, are skipped; equality has only reflexivity and
    symmetry axioms. Resolution uses a per-query time budget (1 s in the FOLIO run).
  - CounterBench: weaker on conditional (92.78%) and backdoor (86.02%) questions, which
    contain observations of derived variables.
  - Runtimes depend on the machine; reported times are from a Colab CPU runtime.

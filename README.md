# Evaluating Generated Code with pass@k

> **Project type:** Coursework / guided evaluation exercise  
> **Status:** Metric demonstrated on a small synthetic example; not a full HumanEval benchmark result

## Overview

This notebook explores how to evaluate generated code using the `pass@k` metric. It loads the OpenAI HumanEval dataset through Hugging Face Datasets, loads the `code_eval` metric through Hugging Face Evaluate, and demonstrates how multiple candidate implementations affect the probability that at least one candidate passes a test.

It also includes an example of generating a Python solution to a FizzBuzz-style problem using Google's Gemini model.

## Problem

A code-generation model may produce several candidate implementations for a programming task. How can we measure whether at least one of the first *k* generated candidates passes the available tests?

The notebook introduces `pass@k` and demonstrates its calculation. The saved result is based on one manually defined addition test case, not on evaluation of the full HumanEval test split.

## Dataset and source

- **Benchmark dataset loaded:** [`openai_humaneval`](https://huggingface.co/datasets/openai_humaneval), accessed through `datasets.load_dataset`.
- **Recorded test split size:** The notebook output shows **164 examples** in the test split.
- **Evaluation library:** Hugging Face `evaluate`, using the `code_eval` metric.
- **Demonstration test case:** `assert add(2,3)==5`.
- **Candidate solutions:** Two hand-written candidate implementations are supplied for the addition task:
  - `def add(a,b): return a*b`
  - `def add(a, b): return a+b`
- **Additional generation example:** A FizzBuzz-style prompt sent to `gemini-pro`.

The HumanEval dataset is loaded in the notebook, but the recorded `pass@k` result is computed on the single custom addition test case shown above. It is not a benchmark score over the 164 HumanEval tasks.

## Contribution and scope

This is a **coursework / guided evaluation exercise**. The notebook demonstrates use of an existing benchmark dataset and evaluation metric; it does not train or fine-tune a code model.

The work represented includes:

- Loading the HumanEval dataset.
- Loading the `code_eval` metric.
- Enabling code evaluation through the `HF_ALLOW_CODE_EVAL` environment variable.
- Supplying multiple candidate implementations for a test case.
- Computing `pass@1` and `pass@2`.
- Prompting Gemini to generate a solution to a FizzBuzz-style programming problem.

## Approach

### 1. Load the benchmark and evaluation metric

```python
from datasets import load_dataset
from evaluate import load

human_eval = load_dataset("openai_humaneval")
code_eval_metric = load("code_eval")
```

The notebook output shows 164 examples in the HumanEval test split.

### 2. Enable code evaluation

```python
import os
os.environ["HF_ALLOW_CODE_EVAL"] = "1"
```

Code evaluation can execute generated programs. Run such evaluations only in a controlled, isolated environment.

### 3. Define a small test case and candidate solutions

```python
test_cases = ["assert add(2,3)==5"]

candidates = [[
    "def add(a,b): return a*b",
    "def add(a, b): return a+b"
]]
```

The first candidate multiplies the inputs and fails the assertion. The second adds the inputs and passes it.

### 4. Compute pass@k

```python
pass_at_k, results = code_eval_metric.compute(
    references=test_cases,
    predictions=candidates,
    k=[1, 2]
)
```

## Results and supported findings

<details>
<summary>Recorded demonstration result</summary>

```text
pass@1 = 0.5
pass@2 = 1.0
```

</details>

For the one addition test case:
- With one candidate, one of the two candidates is correct, giving the displayed `pass@1` of 0.5.
- With both candidates considered, at least one passes, giving the displayed `pass@2` of 1.0.

This illustrates the basic interpretation of pass@k: increasing the number of sampled candidates can increase the chance that at least one candidate passes. It does not prove that a model achieves 100% pass@2 on HumanEval.

The notebook also shows Gemini generating a FizzBuzz-style code snippet. No automated test result for that generated snippet is recorded in the notebook.

## Evaluation methodology

The `code_eval` metric executes candidate code against provided tests and reports pass@k for the supplied candidates.

**Important distinction:** Although the HumanEval test split is loaded, the displayed metric computation uses only one manually authored addition assertion and two manually authored candidate strings. The notebook does not show a loop evaluating model-generated candidates across the HumanEval tasks.

## Limitations

- **Not a full benchmark run:** The recorded score is from one custom test case, not the HumanEval test set.
- **Very small sample:** Two candidates for one task are insufficient for broad conclusions about code-generation quality.
- **Candidates are manually supplied:** The two addition candidates are not shown as outputs sampled from a model.
- **No model comparison:** No systematic comparison across model families, prompt settings, or sampling strategies is recorded.
- **No repeated trials:** Variability across generations is not measured.
- **Limited test coverage:** Passing the single assertion does not establish correctness for all possible inputs.
- **Execution risk:** Code evaluation runs generated code. Use sandboxing, resource limits, and restricted permissions.
- **Environment compatibility:** The notebook sets `HF_ALLOW_CODE_EVAL=1`; dependencies and evaluation behaviour may vary by version.
- **Gemini example is separate:** The FizzBuzz generation cell is illustrative and is not connected to the addition pass@k calculation.

## Reproducibility

Libraries used:

- `datasets`
- `evaluate`
- `huggingface_hub` (as a dependency of the Hugging Face ecosystem)
- `langchain-google-genai` for the Gemini example

Configure any provider credentials outside the notebook. Do not commit API keys.

Suggested layout:

```text
.
├── README.md
├── code_models.ipynb
├── requirements.txt
└── .gitignore
```

## Future improvements

- Run generated candidates against the HumanEval task set rather than a single custom assertion.
- Generate multiple independent candidates per task using documented model versions and sampling parameters.
- Report pass@1, pass@k for several values of *k*, and confidence intervals where appropriate.
- Separate model-generation code from evaluation code.
- Add execution timeouts, memory limits, and sandboxing.
- Preserve evaluation logs and failures for error analysis.
- Compare models under identical prompts, test harnesses, and candidate budgets.

## Disclaimer

This is an educational demonstration of code-generation evaluation. The reported pass@1 and pass@2 values apply only to the single addition test case and two candidate implementations included in the notebook. They must not be presented as HumanEval benchmark scores.

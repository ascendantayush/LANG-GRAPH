# Sequential Workflow — LangGraph

A minimal, end-to-end example of the **sequential graph** pattern in LangGraph: a three-stage
content pipeline that takes raw, unpolished text and returns a localized Hinglish video script.

> This README covers **only this project**. See the repository root README for the rest of the
> LangGraph experiments.

---

## The Pipeline

```
START ──▶ editor ──▶ scriptwriter ──▶ translator ──▶ END
```

| Stage | Node | Reads | Writes | Job |
|:-----:|------|-------|--------|-----|
| 1 | `editor` | `raw_input` | `edited_text` | Fixes grammar, spelling and tone without changing meaning |
| 2 | `scriptwriter` | `edited_text` | `script_text` | Rewrites clean prose as a punchy, spoken video hook |
| 3 | `translator` | `script_text` | `final_output` | Localizes the hook into natural Hinglish |

Every edge is unconditional, so execution is strictly linear — no branching, no loops. That is
the whole point: it is the simplest possible shape a LangGraph pipeline can take, and the base
that conditional routing, fan-out and checkpointing get built on top of.

## Shared State

All three nodes read from and write to one `TypedDict`:

```python
class pipelineSate(TypedDict):
    raw_input: str
    edited_text: str
    script_text: str
    final_output: str
```

A node receives the full state and returns a **partial** dict; LangGraph merges it back in.
Each stage owns exactly one key, which is what makes the nodes independently swappable — the
scriptwriter never sees `raw_input`, so it is never tempted to re-do the editor's work.

## Running It

The notebook is written for Google Colab.

1. Open [`sequential_workflow.ipynb`](sequential_workflow.ipynb) in Colab.
2. Add your Groq API key as a Colab secret named `GROQ_API_KEY`
   (🔑 sidebar → *Add new secret* → toggle notebook access).
3. Run all cells.

**Running locally instead?** Replace the Colab secret lookup

```python
from google.colab import userdata
groq_api_key = userdata.get("GROQ_API_KEY")
```

with an environment variable

```python
groq_api_key = os.environ["GROQ_API_KEY"]
```

and install the dependencies yourself:

```bash
pip install -U langgraph langchain langchain-groq
export GROQ_API_KEY=...
```

## Stack

| Piece | Choice | Why |
|---|---|---|
| Orchestration | `langgraph` | Explicit state + graph wiring |
| Provider | `langchain-groq` | Very high token throughput keeps a 3-hop chain feeling instant |
| Model | `openai/gpt-oss-120b` | Strong instruction-following; handles the Hinglish constraint well |
| Temperature | `0.8` | Two of three stages are creative rewriting; low temp reads flat |

## A Note on the Hinglish Prompt

Stage 3 has by far the longest prompt, and deliberately so. Asked simply to "translate to
Hinglish", models reliably drift into pure Hindi. The prompt therefore states the constraint
explicitly — keep technical vocabulary (AI, coding, data, productivity) in English, mix Hindi
phrasing around it, and never output pure Hindi. This is the one place in the pipeline where
prompt engineering is doing real work rather than just setting a persona.

## Where to Take It Next

- **Conditional edges** — skip translation when the target audience is English.
- **Parallel fan-out** — generate several script variants at once, then pick the best.
- **Checkpointing** — persist state so a run can be paused, inspected and resumed.
- **Human-in-the-loop** — interrupt before the translator and let a person approve the script.

Swap the prompts and the same graph becomes any three-stage pipeline — summarize → outline →
draft, extract → validate → format. The wiring does not change.

## Author

**AYUSH RAJ**

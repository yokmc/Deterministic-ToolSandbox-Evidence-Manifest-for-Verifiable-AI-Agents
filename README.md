# Epion Verifiable AI Demo

This repository is a **minimal, executable demonstration** of verifiable AI agent operations.

It shows how an AI agent can:
- Use tools in a sandboxed environment
- Produce deterministic, replayable behavior
- Automatically generate an Evidence Manifest suitable for audits and compliance

---

## Why this exists

Modern AI systems are powerful but opaque.
This demo focuses on **provability, not performance**.

If an AI system:
- Makes a decision
- Calls tools
- Produces outputs

Then it should be possible to answer:
- What exactly happened?
- Can it be reproduced?
- Can it be audited?

This project answers **yes**.

---

## What this demo includes

- Deterministic ToolSandbox (mocked tools, fixed outputs)
- AI agent using tools (no external access)
- Deterministic replay verification
- Automatic Evidence Manifest generation

---

## Project structure

```

agent.py        # AI agent logic
toolsandbox.py  # Deterministic tool environment
trace.py        # Trace recording & replay check
manifest.py     # Evidence Manifest generation
run_demo.py     # End-to-end execution

````

---

## How to run

```bash
python run_demo.py
````

Expected output:

```
Replay OK: True
Evidence Manifest generated → evidence.json
```

---

## Evidence Manifest

After execution, an `evidence.json` file is generated automatically.

This file acts as:

* Audit evidence
* Compliance artifact
* Trust proof for AI operations

---

## Design principles

* Reproducibility over randomness
* Evidence over claims
* Governance over raw performance

---

## Intended audience

* AI platform teams
* Security & compliance teams
* Enterprise architects
* Investors evaluating AI governance infrastructure

---

## License

MIT (for demo and reference purposes)

````

---

# 🧩 코드 파일들 (복붙용)

### `toolsandbox.py`
```python
import hashlib
import json

class ToolSandbox:
    def __init__(self):
        self.trace = []

    def _record(self, tool, input_data, output_data):
        record = {
            "tool": tool,
            "input": input_data,
            "output": output_data,
            "hash": hashlib.sha256(
                json.dumps(output_data, sort_keys=True).encode()
            ).hexdigest()
        }
        self.trace.append(record)
        return output_data

    def get_orders(self, customer_id):
        orders = [
            {"order_id": 1, "amount": 120},
            {"order_id": 2, "amount": 80}
        ]
        return self._record(
            "GET:/orders",
            {"customer_id": customer_id},
            orders
        )

    def write_report(self, summary):
        return self._record(
            "WRITE:report.txt",
            {},
            {"content": summary}
        )
````

---

### `agent.py`

```python
class Agent:
    def __init__(self, sandbox):
        self.sandbox = sandbox

    def run(self, customer_id):
        orders = self.sandbox.get_orders(customer_id)
        total = sum(o["amount"] for o in orders)
        summary = f"Customer {customer_id} total amount: {total}"
        self.sandbox.write_report(summary)
```

---

### `trace.py`

```python
import json

def save_trace(trace, path):
    with open(path, "w") as f:
        json.dump(trace, f, indent=2)

def replay(trace1, trace2):
    return trace1 == trace2
```

---

### `manifest.py`

```python
import json
from datetime import datetime

def generate_manifest(trace, replay_ok):
    manifest = {
        "manifest_version": "1.0",
        "verifiers": [
            {
                "name": "ToolSandbox",
                "predicate": "pass" if replay_ok else "fail",
                "evidence": {
                    "steps": len(trace),
                    "replay": replay_ok
                }
            }
        ],
        "overall": "pass" if replay_ok else "fail",
        "timestamp": datetime.utcnow().isoformat() + "Z"
    }

    with open("evidence.json", "w") as f:
        json.dump(manifest, f, indent=2)

    return manifest
```

---

### `run_demo.py`

```python
from toolsandbox import ToolSandbox
from agent import Agent
from trace import save_trace, replay
from manifest import generate_manifest

# Run #1
sandbox1 = ToolSandbox()
agent1 = Agent(sandbox1)
agent1.run(customer_id=123)
trace1 = sandbox1.trace
save_trace(trace1, "trace_run1.json")

# Run #2 (deterministic replay)
sandbox2 = ToolSandbox()
agent2 = Agent(sandbox2)
agent2.run(customer_id=123)
trace2 = sandbox2.trace
save_trace(trace2, "trace_run2.json")

# Verify deterministic replay
replay_ok = replay(trace1, trace2)

# Generate evidence
generate_manifest(trace1, replay_ok)

print("Replay OK:", replay_ok)
print("Evidence Manifest generated → evidence.json")
```

---



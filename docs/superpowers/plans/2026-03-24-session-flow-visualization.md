# Session Flow Visualization — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an interactive Canvas 2D flow visualization with holographic/sci-fi aesthetics to session replay pages, showing agent hierarchy, tool usage, and message flow.

**Architecture:** Python builds a flow graph from the existing flat message list, embeds it as JSON in session HTML. A self-contained `SessionFlow` JS class renders the graph on a Canvas 2D element with force-directed layout, animated particles, and auto-play timeline. The canvas sits above the existing chat panel in a 40/60 split.

**Tech Stack:** Python 3.8+ (graph construction), Vanilla JS (Canvas 2D rendering), inline CSS (layout/responsive)

**Spec:** `docs/superpowers/specs/2026-03-24-session-flow-visualization-design.md`

---

## File Map

| File | Action | Responsibility |
|------|--------|----------------|
| `extract_stats.py` | Modify | Add `build_session_flow()`, modify `generate_session_pages()` and `_get_session_html_template()` |

All changes are in the single file `extract_stats.py` — consistent with the existing monolith approach. The session HTML template, CSS, and JS are all inline strings within this file.

**JS class method insertion:** Tasks 4-8 add methods to the `SessionFlow` class created in Task 3. Insert each new method immediately before the closing `}` of the class (before the `// Initialize if flow data` comment). Methods within the class are unordered so position doesn't matter — just keep them inside the class body.

---

### Task 1: Python — `build_session_flow()` function

**Files:**
- Modify: `extract_stats.py` (insert new function before `generate_session_pages()` at ~line 3480)

This function takes the flat message list from `extract_session_messages()` and produces the flow graph data structure.

- [ ] **Step 1: Write `build_session_flow()` function**

Insert before `generate_session_pages()` (~line 3480):

```python
def build_session_flow(messages):
    """Build a flow graph from the flat message list for Canvas visualization."""
    if not messages:
        return {"agents": [], "events": [], "edges": []}

    # Main agent is always present
    agents = [{
        "id": "main",
        "name": "Claude",
        "type": "main",
        "parent_id": None,
        "tokens": {"input": 0, "output": 0, "cache_read": 0, "cache_write": 0},
        "cost": 0.0,
        "tools_summary": {}
    }]
    events = []
    edges = []
    subagent_counter = 0

    # Determine session start time for relative timestamps
    # Note: using the first message with a valid timestamp as the baseline,
    # not necessarily index 0, since some messages may lack timestamps.
    first_ts = None
    for m in messages:
        ts = m.get("timestamp")
        if ts:
            if isinstance(ts, str):
                try:
                    from datetime import datetime
                    first_ts = datetime.fromisoformat(ts.replace("Z", "+00
```

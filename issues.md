# Issues Found and Fixed — `society_dues.py`

Line numbers refer to the **original** file. Tool: **B** = Bandit, **P** = Pylint, **F** = Flake8.
Ordered by severity (high/medium first).

| # | Issue | Tool / Code | Type | Line(s) | Description | Fix Approach |
|---|-------|-------------|------|---------|-------------|--------------|
| 1 | SQL injection | B608 (Medium) | Security | 9 | `addDue` builds the INSERT with an f-string, so `flat`/`month` such as `x'); DROP TABLE dues;--` change the query. | Use a parameterised query: `conn.execute("INSERT ... VALUES (?, ?, ?, 0)", (flat, month, amount))`. |
| 2 | Use of `eval()` on input | B307 (Medium) / P W0123 | Security | 21 | `penalty(rule)` runs `eval` on a string, allowing arbitrary code execution, e.g. `__import__('os').system(...)`. | Parse with `ast.parse(..., mode="eval")` and evaluate only numeric constants and `+ - * / // %` through a whitelist (`_safe_eval`). Anything else returns 0. |
| 3 | Hard-coded password | B105 (Low) / CWE-259 | Security | 5 | `TREASURER_PASSWORD = "flat#101"` is stored in source and in git history. | Read it from the environment: `os.environ.get("TREASURER_PASSWORD", "")`. |
| 4 | Mutable default argument | P W0102 | Bug | 8 | `history=[]` is created once, so every call to `addDue` appends to the same shared list. | Default to `None` and create a new list inside the function. |
| 5 | Overly broad exception | P W0718 | Quality | 22 | `except Exception` hides unrelated bugs. | Catch `(ValueError, SyntaxError, ZeroDivisionError)` only. |
| 6 | Inconsistent return | P R1710 | Quality | 12-17 | `collect` returns `row[0]` or implicitly `None`. | Add an explicit `return None` and document it. |
| 7 | Comparison to `None` with `!=` | F E711 / P C0121 | Style | 29 | `rows != None` should use identity. `fetchall()` never returns `None` anyway. | Removed the dead check and return the list comprehension directly. |
| 8 | Function name not snake_case | P C0103 | Style | 8 | `addDue` violates PEP 8 naming. | Renamed to `add_due` and updated the call sites. |
| 9 | Missing docstrings | P C0114 / C0116 | Documentation | 1, 8, 12, 19, 27, 34 | The module and five functions have no docstrings. | Added a module docstring and one for every function. |
| 10 | Lines too long | F E501 / P C0301 | Style | 4, 13, 15 | Lines of 87, 104 and 92 characters (limit 79). | Split SQL strings and arguments across lines. |
| 11 | Missing blank lines | F E302 | Style | 12 | No blank lines between `addDue` and `collect`; PEP 8 needs two. | Added two blank lines. |
| 12 | Code runs on import | Best practice | Quality | end of file | `main()` was called at module level. | Wrapped it in `if __name__ == "__main__":`. |

## Results

| Tool | Before | After |
|------|--------|-------|
| Bandit | 3 issues (0 High, 2 Medium, 1 Low) | 0 issues |
| Flake8 | 5 violations | 0 violations |
| Pylint | 5.81 / 10 | **10.00 / 10** |

The demo output is unchanged: `2500`, `100`, `['A-102']`.

## Commits

1. `Fix Bandit findings` (rows 1, 2, 3, 5)
2. `Fix Flake8 issues` (rows 7, 10, 11)
3. `Fix Pylint issues` (rows 4, 6, 8, 9, 12)

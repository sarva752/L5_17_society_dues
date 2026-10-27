# Reflection — Lab 5: Static Code Analysis

**1. Which issues were the easiest and which were the hardest to fix?**
The easiest were the style and documentation items (Flake8 blank lines and line length, Pylint docstrings, renaming `addDue` to `add_due`). They are mechanical and the tool says exactly what to change. The hardest was the `eval()` in `penalty`. Deleting `eval` was not enough because the program needs to compute `"50 * 2"`, so I had to write a small AST-based evaluator that allows only numbers and arithmetic operators. The SQL injection fix was easy to apply but needed me to understand why `?` placeholders work: the database treats the values as data, not as part of the SQL.

**2. Did the tools report any false positives?**
Nothing was clearly false, but some findings needed judgement. Bandit's B105 flagged `"flat#101"` as a hard-coded password. It is a real concern, but it only fires because the variable name contains "PASSWORD", so a secret with a different name would likely be missed. Pylint's `W0718` on `except Exception` was valid, although in a top-level handler that logs the error it can be acceptable. If a finding were truly intentional, I would suppress it narrowly with a comment such as `# nosec B105` and explain why, not disable the check globally.

**3. How would you integrate static analysis into a real development workflow?**
I would run Flake8, Pylint and Bandit as pre-commit hooks so problems are caught before they enter history. I would also run them in a CI pipeline (e.g. GitHub Actions) on every pull request and fail the build on any Bandit Medium/High finding or on a Pylint score below a set threshold. A shared config file (`.pylintrc`, `setup.cfg`/`.flake8`) would keep all developers on the same rules.

**4. What tangible improvements did I see?**
Security: SQL injection and arbitrary code execution through `eval` are closed, and the password is no longer in source control. Correctness: the shared mutable `history` list bug is gone, and `collect` has a well-defined return value. Maintainability: the code is PEP 8 compliant, documented and importable without side effects. Pylint went from 5.81 to 10.00, Bandit from 3 findings to 0 and Flake8 from 5 to 0, and the program output is identical. The tools also found a real bug (the mutable default) that is easy to miss in manual testing because the demo only calls `addDue` without `history`.

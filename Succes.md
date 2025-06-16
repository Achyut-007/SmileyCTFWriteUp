## Success (Fully Solved)

**Summary:**  
- Given a Haskell program with a flag length of 39 chars.
- Flag must satisfy mixed arithmetic and bitwise constraints.

**How I Solved It:**  
- Learned Z3 SMT solver.
- Modeled each char as symbolic integer.
- Converted Haskell constraints to Z3.

**Key Z3 Example:**
```python
from z3 import *

chars = [Int(f'c{i}') for i in range(39)]
s = Solver()

for c in chars:
    s.add(c >= 32, c <= 126)

# Example constraints
s.add(chars[37] * chars[15] == 3366)
s.add(BitOr(chars[6], chars[37]) == 105)

if s.check() == sat:
    m = s.model()
    flag = ''.join(chr(m[c].as_long()) for c in chars)
    print(flag)
else:
    print("No solution found.")
```

**Result:**  
- Solver found a valid flag satisfying all constraints.
- Learned Z3 for future CTFs!


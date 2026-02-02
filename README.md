# CC0 Circuit Construction

Code for [Udeshi & Chapman, MIT PRIMES-USA 2025](https://math.mit.edu/research/highschool/primes/materials/2025/Udeshi-Chapman.pdf).

It uses the [Z3](https://github.com/Z3Prover/z3) SMT solver to find the **smallest constant-depth circuits built only from MOD_m gates** (CC⁰[m] circuits) that compute a given Boolean function.

## The model

- The circuit has `n` input gates `x0 … x(n-1)` and `s` MOD_m gates.
- Each MOD_m gate takes a weighted sum of a constant and of earlier gates, with weights in `{0, …, m-1}`. It outputs **1 exactly when that sum ≡ 0 (mod m)**.
- Each gate's depth is one more than the depth of its deepest input. The circuit's depth is capped by `max_d`.
- The last gate is the output, and it must match the target truth table on all `2^n` inputs.
- The whole search is encoded as one bit-vector (`QF_BV`) problem. The gate count `s` goes up by one at a time until Z3 finds a circuit, so the first circuit found has the fewest gates possible within the depth cap.
- To skip equivalent copies of the same circuit, two symmetry-breaking rules are added:
  - Gates on the same layer must appear in lexicographic order.
  - Each gate's first nonzero weight must be a proper divisor of `m`.

## Files

| File | Paper | What it does |
|---|---|---|
| `cc0_first_model.py` | Appendix A.1 | Finds **one** smallest circuit and prints it |
| `cc0_enumeration.py` | Appendix A.2 | Finds **every** minimum-size circuit. Circuits that differ only by relabeling gates are counted once. |

## Running it

```bash
pip install z3-solver
python cc0_first_model.py
python cc0_enumeration.py
```

To change what gets searched, edit the block at the bottom of either file:

```python
f = [int(bin(i).count("1") % 4 == 0) for i in range(1 << 7)]  # target truth table: MOD4 on 7 bits
m = 5        # gate modulus (MOD_5 gates)
max_s = 5    # largest number of MOD gates to try
max_d = 2    # depth limit
```

- `f` must have length `2^n`. In entry `i`, `x0` is the most significant bit of `i`.
- `cc0_first_model.py` starts its search at `s = 4` gates, so it never tries circuits with fewer. `cc0_enumeration.py` starts at `s = 1`.

## Example output

Running `cc0_first_model.py` as shipped (MOD4 on 7 bits, MOD_5 gates, depth ≤ 2) takes about 20 seconds:

```
circuit with 4 gates (depth<=2)

MOD gates:
  Gate 7 (depth 1): MOD_5(g0, g1, 2*g2, g3, g4, g5, 3*g6)
  Gate 8 (depth 1): MOD_5(g0, g1, 4*g2, g3, g4, g5, 4*g6)
  Gate 9 (depth 1): MOD_5(g0, 4*g1, g3, g4, g5, g6)
  Gate 10 (depth 2): MOD_5(1, g0, g1, 3*g2, g3, g4, g5, g6, 3*g7, 3*g8, 3*g9)

output at gate 10 with depth 2
```

Each gate lists its weighted inputs. `3*g7` means weight 3 on gate 7, and `1` is the constant term.

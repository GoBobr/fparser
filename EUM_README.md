# EUM Fork of fparser

This is an EUMETSAT fork of [stfc/fparser](https://github.com/stfc/fparser),
maintained on the `eum-fortran` branch for use in the CO2M mission Fortran
static analysis pipeline ([fortran-ast-checker](https://github.com/GoBobr/fortran-ast-checker)).

## Branch Structure

| Branch | Description |
|--------|-------------|
| `master` | Mirror of upstream `stfc/fparser` master |
| `eum-fortran` | EUM feature branch with EUM-specific fixes (based on `v0.2.4`) |
| `upstream/*` | Mirror of upstream feature branches |

## EUM Changes (relative to v0.2.4)

### Fix: Unary minus after multiplication/division operators

fparser 0.2.4 cannot parse expressions where a unary minus follows a
multiplication or division operator:

```fortran
a * -b
a + d * -b
dtaus_all(ik, ix)*-((taus_aer + taus_ray)**(-2))
```

The Fortran standard permits this — the `-` is a level-2 unary expression —
but fparser has two limitations that are fixed in this branch:

1. **`Mult_Operand.subclass_names`** only includes `Level_1_Expr`, so it
   cannot match `-b` as a right-hand operand of `*`.
   **Fix**: Added `Level_2_Unary_Expr` to `Mult_Operand.subclass_names`.

2. **`Level_2_Expr.match`** uses `BinaryOpBase.match` which splits the
   expression string at every `[+-]` occurrence using `rsplit()`. In
   `a + d * -b` the rightmost `-` (unary minus) is treated as a binary
   subtraction operator, causing the split `lhs="a + d *"`, `rhs="b"` —
   which fails because `a + d *` is not a valid expression.
   **Fix**: Override `Level_2_Expr.match` to rewrite `*-` / `/-` as
   `*(-1)*` / `/(-1)/` before calling `BinaryOpBase.match`. This is
   mathematically equivalent and only affects the internal string
   representation, not the AST semantics.

All 206 existing Fortran2003 tests pass with no regressions.

## Syncing with Upstream

```bash
git fetch upstream
git checkout eum-fortran
git rebase upstream/master  # or merge
git push origin eum-fortran
```

## Installation

```bash
pip install git+https://github.com/GoBobr/fparser.git@eum-fortran
```

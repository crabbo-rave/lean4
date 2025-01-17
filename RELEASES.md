v4.0.0-m4 (WIP)
---------

* Support notation `let <pattern> := <expr> | <else-case>` in `do` blocks.

* Remove support for "auto" `pure`. In the [Zulip thread](https://leanprover.zulipchat.com/#narrow/stream/270676-lean4/topic/for.2C.20unexpected.20need.20for.20type.20ascription/near/269083574), the consensus seemed to be that "auto" `pure` is more confusing than it's worth.

* Remove restriction in `congr` theorems that all function arguments on the left-hand-side must be free variables. For example, the following theorem is now a valid `congr` theorem.
```lean
@[congr]
theorem dep_congr [DecidableEq ι] {p : ι → Set α} [∀ i, Inhabited (p i)] :
                  ∀ {i j} (h : i = j) (x : p i) (y : α) (hx : x = y), Pi.single (f := (p ·)) i x = Pi.single (f := (p ·)) j ⟨y, hx ▸ h ▸ x.2⟩ :=
```

* [Partially applied congruence theorems.](https://github.com/leanprover/lean4/issues/988)

* Improve elaboration postponement heuristic when expected type is a metavariable. Lean now reduces the expected type before performing the test.

* [Remove deprecated leanpkg](https://github.com/leanprover/lean4/pull/985) in favor of [Lake](https://github.com/leanprover/lake) now bundled with Lean.

* Various improvements to go-to-definition & find-all-references accuracy.

* Auto generated congruence lemmas with support for casts on proofs and `Decidable` instances (see [whishlist](https://github.com/leanprover/lean4/issues/988)).

* Rename option `autoBoundImplicitLocal` => `autoImplicit`.

* [Relax auto-implicit restrictions](https://github.com/leanprover/lean4/pull/1011). The command `set_option relaxedAutoImplicit false` disables the relaxations.

* `contradiction` tactic now closes the goal if there is a `False.elim` application in the target.

* Renamed tatic `byCases` => `by_cases` (motivation: enforcing naming convention).

* Local instances occurring in patterns are now considered by the type class resolution procedure. Example:
```lean
def concat : List ((α : Type) × ToString α × α) → String
  | [] => ""
  | ⟨_, _, a⟩ :: as => toString a ++ concat as
```

* Notation for providing the motive for `match` expressions has changed.
Before:
```lean
match x, rfl : (y : Nat) → x = y → Nat with
| 0,   h => ...
| x+1, h => ...
```
Now:
```lean
match (motive := (y : Nat) → x = y → Nat) x, rfl with
| 0,   h => ...
| x+1, h => ...
```
With this change, the notation for giving names to equality proofs in `match`-expressions is not whitespace sensitive anymore. That is,
we can now write
```lean
match h : sort.swap a b with
| (r₁, r₂) => ... -- `h : sort.swap a b = (r₁, r₂)`
```

* `(generalizing := true)` is the default behavior for `match` expressions even if the expected type is not a proposition. In the following example, we used to have to include `(generalizing := true)` manually.
```lean
inductive Fam : Type → Type 1 where
  | any : Fam α
  | nat : Nat → Fam Nat

example (a : α) (x : Fam α) : α :=
  match x with
  | Fam.any   => a
  | Fam.nat n => n
```

* We now use `PSum` (instead of `Sum`) when compiling mutually recursive definitions using well-founded recursion.

* Better support for parametric well-founded relations. See [issue #1017](https://github.com/leanprover/lean4/issues/1017). This change affects the low-level `termination_by'` hint because the fixed prefix of the function parameters in not "packed" anymore when constructing the well-founded relation type. For example, in the following definition, `as` is part of the fixed prefix, and is not packed anymore. In previous versions, the `termination_by'` term would be written as `measure fun ⟨as, i, _⟩ => as.size - i`
```lean
def sum (as : Array Nat) (i : Nat) (s : Nat) : Nat :=
  if h : i < as.size then
    sum as (i+1) (s + as.get ⟨i, h⟩)
  else
    s
termination_by' measure fun ⟨i, _⟩ => as.size - i
```

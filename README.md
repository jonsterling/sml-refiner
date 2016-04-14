This repository contains a collection of very modular libraries for building
tactic-based refiners in the LCF tradition.

### LCF: modernized refinement proof

We begin with a modernized version of the ancient LCF signature, with a few
changes.

#### terminology

First of all, terminology has been updated to reflect modern usage and avoid
conflation of distinct notions. In short, you provide a type of `judgments` and
a type of `evidence`, and the signature specifies the right sort of apparatus
to build a refinement logic that proves judgments by synthesizing evidence.
Subgoals are collected in a `ctx`, and a `evidence ctx` is called an
`environment`. A function `environment -> evidence` is called a `validation`,
and represents *hypothetical (admissible) evidence*.

In Classic LCF, we have `type 'a ctx = 'a list`.

#### (relative) monadic design

For a type `'a`, we leave it up to a particular implementation of LCF to decide
what it means for `'a` to suffice as a judgment; `'a` is equipped with this
apparatus by means of the `'a Judgable.t` functor. In Classic LCF, this is the
identity, but in Dependent LCF, it induces a classification of evidence by
valence and a substitution action for evidence.

We have decomposed the LCF `tactic` type by means of an auxiliary type `'a
state` of proof states with judgments in `'a`. Then, the standard tree
operations for LCF tactic proofs (such as the classic `THEN*` tacticals) are
captured by the Kleisli extension operation of a relative monad on the functor
`'a Judgable.t`.

#### correctness conditions

The correctness conditions of LCF refiners are usually said to be based on
having an abstract type of proofs ("evidence") that are synthesized, but this
is not part of the essence of LCF; indeed, in a system like Nuprl it is crucial
that this type not be abstract, since the synthesis of truth judgments is
simply another Nuprl term, which can be constructed by hand from anywhere.
Therefore, we may not in general view an LCF refiner as being in the business
of synthesizing proof: *proof is an act, not an object*.

[Randy Pollack](http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.29.9573)
includes the following in his list of ways a refiner may go wrong:

> It might return the wrong theorem [evidence]. ... We must execute a
> validation to completion and examine the `thm` [`evidence`] object it returns
> to be sure it is the intended one.

This is not in fact correct at all, at least, not from our perspective: a
refiner is a *definition* of a formal proof theory, and as such, whatever the
validation returns is the correct evidence *by definition*. Contrary to the
approach that has been taken with the Coq proof assistant, we intend a refiner
to be the definition of the proof theory, not a means to synthesize terms which
may be checked by a type checker, which is itself definitive.

The approach that we advocate scales better to larger objects, and is also
easier to verify: rules written for a refiner are usually very similar to how
they appear on paper, and it is easy to tell when they are correct. In
contrast, typecheckers are written at a far lower level of abstraction and are
more difficult to verify. It is clear that making the refiner definitive (and
thence trusted) is preferable to making the typechecker definitive (and thence
trusted), both in respect of correctness and in respect of real-world performance.


### Dependent LCF

Dependent LCF is an implementation of the LCF interface which provides a
*principled* way to deal with dependent refinement, where *evidence* of one
goal plays a part in the *statement* of another goal. Subgoals are collected,
then, into *telescopes* rather than lists.

Unlike standard approaches, including that used by Coq's refiner, the
correctness conditions of the dependent refinement are completely local to the
Dependent LCF library; that is, we provide machinery for dependent refinement
that is valid regardless of the logic it is deployed at, and this comes
crucially from the following facts:

1. Metavariables correspond one-to-one with goals. There is no difference
   between a hole and a goal.

2. A metavariable can be solved *only* by applying a refinement rule to the
   judgment with which it is labeled. Dependent LCF does not solve
   metavariables by unification, and there is no non-local solution of
   metavariables.

In contrast, standard approaches to dependent refinement often involve
existential variables which are not intrinsically linked to a single goal, and
which can be solved by unification and by non-local action. This is very
convenient for proof automation, but it cannot be built into a *general-purpose*
refinement framework.

Consider the case of a logic with subtyping or polymorphism: such extensions
would allow invalid refinement steps, meaning that the synthesis of refinement
cannot be trusted and must be verified separately, which is manifestly what we
intend to avoid in *defining* a logic by its refiner.  We believe that whatever
affordances for metavariables are needed (e.g. non-local solution, etc.) can be
built up as rules in an individual refiner with no loss of expressivity.

The limitation and strictures imposed here are necessary because, rather than
building a refiner for a particular logic, we intend to build a refinement
which can be used for defining arbitrary logics. As such, our constraints are
very different from those that led to the current design of Coq's refiner, and
so we have arrived at a different approach.

### Nominal LCF

Nominal LCF is a concrete language for refinement proof with well-scoped
hypothesis names. This is a non-trivial problem, because the binding of fresh
hypothesis names (e.g. as generated by a *left rule* in a sequent calculus) is
dependent on the goal at which a tactic is run. As a result, this is not a
static property, but a dynamic one.

Nominal LCF addresses this problem by construing tactics as *continuously*
consuming a free choice sequence of atoms/symbols. A free choice sequence is
like an infinite stream, except its elements are got by interaction with a
subject (i.e. a user) rather than by means of a computable function.

The continuity of nominal tactics allows us to calculate how much of the name
sequence a particular tactic has consumed (i.e. compute the modulus of
continuity of the stream processor), and use this in order to split the
sequence between tactics in order to ensure that names are consumed in the
right places when the script is run.

Nominal LCF may be elaborated into any LCF structure, including Classic LCF and
Dependent LCF.

-----------------------------------------------------------------------------------

### Instructions

```
git submodule init --recursive
rlwrap sml
> CM.make "development.cm";
```

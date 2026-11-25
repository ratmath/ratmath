# Library roadmap: what is left, and five further packages (design)

Status: proposal, written 2026-11-25. Nothing in sections 3 to 7 is built.
It follows the gold-plating pass of 2026-11-25 (linalg#4, analytic#9,
algebraic#5, graphics#7, all merged) and answers four questions:

1. What from the gold-plating document was not done?
2. What would a complex package, a number theory package and an abstract
   algebra package contain? (A statistics package and a discrete mathematics
   package were added to the plan on review, sections 6 and 7.)
3. How much of an undergraduate mathematics curriculum do the libraries cover
   today, and how much would those packages add?
4. What would still remain?

Line counts for future work are estimates made from the size of comparable
code already in the libraries. They are for ordering the work, not promises.
Curriculum percentages in section 8 are a judgement, explained there.

## 1. The map today

There are two families of code, and this document is mostly about the first.

**The RatMath Shelf: the JavaScript libraries** (each its own repository, Bun and ES modules, built on
`@ratmath/core`, answers come as a fast preview followed by an exact or
certified value):

| Package | Source lines | What it covers |
|---|---|---|
| `@ratmath/core` | | Integers, rationals, intervals, regions, the `R` template, numeral systems, the shared `Refinement` |
| `@ratmath/analytic` | 8,900 | Taylor models with proven remainders; elementary and special functions; limits; integrals (proper, improper, several variables, line and surface); zeros, poles, extrema; piecewise functions; Fourier series; ODE initial and boundary value problems |
| `@ratmath/algebraic` | 5,000 | Real and complex algebraic numbers; factoring over Q, number fields and finite fields; splitting fields; Galois groups to degree 7 with the correspondence; permutation groups; resultants, partial fractions; continued fractions and Pell; modular arithmetic; LLL |
| `@ratmath/linalg` | 4,400 | Exact solving and every standard factorisation and canonical form; eigenvalues as algebraic numbers; matrix functions; inexact entries with certified results; workers; teaching forms |
| `@ratmath/graphics` | 6,500 | Certified graphs and their features; parametric, polar, implicit; exact geometry and regions (conics exactly); calculus pictures; surfaces and space curves; animation; SVG, PNG, PDF, TikZ |
| `ratmath/rust` | | Rust kernels (WASM and C ABI) for the hot loops |

**RiX plugins** (in `ratmath/rix`, written mostly in RiX itself, loaded with
`.Plugin.Load`). These already reach further across the curriculum than the
JavaScript libraries do, at a smaller depth each. The ones that matter for this
document, from their READMEs:

| Plugin | What it has |
|---|---|
| `complex` | A complex number as two certified reals; arithmetic, conjugate, parts. Separate from core `.Complex` (exact, generator `i`) and `.ball.Complex` (rectangles) |
| `cayley` | A Cayley–Dickson kernel for dimension 2^n over any scalar backend; inverse only where the algebra allows it |
| `quaternion`, `octonion` | Typed fronts over `cayley`. Exact arithmetic, norms, inverses, left and right division. `Exp`, `Log`, roots and trig exist but are float-backed and marked approximate |
| `exact-algebras` | The older rational quaternion and octonion values |
| `complex-visualization` | Domain colouring and four-coordinate slices of rational functions |
| `probability`, `stats` | Finite probability, the named distributions, Bayes; descriptive statistics, the standard tests and confidence intervals, with interval-valued variants |
| `combinatorics`, `graph`, `logic` | Lazy enumeration; weighted graphs and shortest paths with evidence; propositional logic, truth tables, normal forms, sequents |
| `poly`, `ratfun`, `algebra`, `cas`, `symbolic`, `calculus`, `analysis`, `cauchy` | One-variable polynomials and rational functions, a bounded CAS, course calculus |

So James's recollection is right: RiX has complex numbers, quaternions and
octonions. What RiX does not have is certified complex *analysis* (the
elementary functions of its quaternions are floats, and `WORK_PLAN.md` lists
"complex special functions/contour integration/root isolation" under later
item D11 and named branches under D3).

The family of subject libraries is called the **RatMath Shelf** (James,
2026-11-25). A separate change (ratmath/ratmath#3) pins every Shelf library
as a submodule of this repository under `shelf/`, so that this repository
oversees them. The five packages below would join the Shelf there as they
appear.

## 2. Left over from the gold-plating document

Everything in that document that was not built, with where it should go.

| Item | Estimate | Where it goes |
|---|---|---|
| **Linalg** | | |
| Symbolic parameters (determinant of a matrix in `t`) | 400 for polynomial entries, 1,500 in full | Polynomial entries can be done in linalg now. The full version needs several-variable polynomials, which section 5 provides |
| Complex and algebraic entries inside `Matrix` | 700 | Linalg, after the complex package settles the exact complex type (section 3) |
| Sparse and banded storage | 800 | Linalg, when a use needs it |
| Rust kernels for det, solve, charpoly wired into linalg | 300 of glue | Linalg and `ratmath/rust` |
| Faster real-root isolation for large characteristic polynomials | 400 | Algebraic (Descartes or continued-fraction isolation) |
| Matrix log and roots with eigenvalues on the negative real axis | small | Linalg, after the complex package defines branches |
| **Algebraic** | | |
| Integral bases and field discriminants | 600 | Number theory package (section 4) |
| Class numbers and unit groups | 1,500 | Number theory package |
| Elliptic curves over Q and prime fields | 1,200 | Number theory package |
| p-adic numbers | 500 | Number theory package |
| Conway polynomials | 150 plus a 20 kB table | Abstract algebra package (section 5), or algebraic's finite fields |
| Galois groups of degree 8 and up; tested examples for 6T10 and A6 | 600 | Algebraic |
| Fast `compare` of an algebraic number with a rational of huge denominator | 100 | Algebraic (sign of the polynomial at the rational). Graphics works around this today |
| **Analytic** | | |
| Complex analytic functions | 1,500 in the document; more if done properly | Complex package (section 3) |
| Limits needing cancellation of leading terms; conditionally convergent improper integrals | 600 | Analytic |
| Speed of surface integrals and nonlinear boundary value problems | | Analytic |
| **Graphics** | | |
| Geometry editor | 1,800 | An app, not the library (James, 2026-11-25) |
| Exact arcs for cubic and higher boundaries; `implicit()` exact for conics | 900 | Graphics; the cubic case becomes easier with section 5's elimination |
| Features of parametric curves (self-intersections, cusps, arc length) | 500 | Graphics |
| Hidden-line removal for self-intersecting surfaces | 400 | Graphics |
| **Across packages** | | |
| npm dependencies in place of git submodules in linalg and graphics | small | Needs algebraic and analytic published to npm first |
| calc-app using linalg's `field` module | small | calc-app |

## 3. A complex package: `@ratmath/complex`

James's decision of 2026-11-25 was that complex analysis does not belong in
analytic because "the complex world seems rather different in character". This
package is where it goes, together with the hypercomplex numbers.

### 3.1 Scope

**Numbers.**
- Exact complex rationals (Gaussian rationals) with the same parsing and
  display conventions as Core.
- Enclosures: rectangles and discs with rational data, as `Refinement`s.
- Polar form with certified modulus and argument; exact roots of unity and
  the exact trigonometric values come from algebraic.
- Conversions to and from algebraic's `ComplexAlgebraic`, which stays where it
  is (it is a root of a polynomial, not an enclosure).

**Elementary functions with honest branches.** `exp`, `log`, powers, roots,
trigonometric and hyperbolic functions and their inverses on enclosures. The
rule already written in `WORK_PLAN.md` D3 applies: never assume the principal
branch silently. A value whose enclosure touches a cut is reported as
straddling it, with both candidate values, and a branch can be named
explicitly or carried along a path (continuation).

**Analysis.**
- Complex Taylor models: a polynomial with complex rational coefficients plus
  a proven remainder on a disc. This is analytic's central idea with a
  different coefficient field, so the first design question is whether
  analytic's `TM` can take its coefficient arithmetic as a parameter or
  whether the complex one is written separately (see 3.3).
- Contour integrals along segments, arcs and user paths, certified.
- The argument principle as a *proof*: the number of zeros minus poles inside
  a contour is an integer, so an enclosure narrower than 1 decides it. From
  that, certified isolation of all zeros of an analytic function in a region,
  which analytic cannot do on the real line for roots of even multiplicity.
- Residues, Laurent coefficients, classification of isolated singularities,
  and real integrals evaluated by residues with the steps shown.
- Möbius transformations (exact over the complex rationals), cross-ratio,
  the Riemann sphere, and conformal maps of standard regions.
- Later: gamma and zeta on the complex plane, which the number theory package
  would use.

**Hypercomplex numbers.** A Cayley–Dickson construction over any scalar:
quaternions, octonions, sedenions.
- Exact arithmetic, conjugates, norms, inverses where they exist, left and
  right division, and the identities each level keeps or loses (commutative,
  associative, alternative, zero divisors at dimension 16).
- Quaternions as rotations: exact rational unit quaternions, conversion to and
  from rotation matrices (linalg) and interpolation, which graphics' 3D views
  can use.
- Integer quaternions (Lipschitz and Hurwitz) for the four-squares theorem,
  shared with the number theory package.
- Certified `exp`, `log` and roots on the slice through a given quaternion or
  octonion, replacing the float versions RiX has now.

**Pictures** live in graphics, called the same way analytic is today: domain
colouring, the image of a grid under a map, contours with their zeros and
poles marked, and Riemann sphere views.

### 3.2 Relation to RiX

RiX's `complex`, `cayley`, `quaternion` and `octonion` plugins are written in
RiX and tied to its evidence records. They stay independent of this package
for now (James, 2026-11-25): the RiX plugins exist to exercise the language
itself, so rewriting them as calls into JavaScript would defeat their purpose.
The same holds for the other packages in this document and their RiX
counterparts.

What should grow instead is the bridge between RiX and the Shelf
(`ratmath/rix-bridge`, which already exposes analytic, linalg, algebraic and graphics to RiX as
plugins), so that a RiX user can reach each new package directly when the certified version of something is wanted, for example
contour integration or the elementary functions of a quaternion, without the
plugin depending on it.

### 3.3 Dependencies and phases

Depends on core and analytic (real enclosures of `exp`, `sin`, `atan`, the
constants), and optionally on algebraic for conversions.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Exact complex rationals, rectangles and discs, polar form, elementary functions with branch reporting | 1,400 |
| 2 | Cayley–Dickson numbers, quaternion rotations, integer quaternions | 700 |
| 3 | Complex Taylor models; contour integrals; argument principle and zero isolation | 2,200 |
| 4 | Residues, Laurent series, singularities, real integrals by residues | 800 |
| 5 | Möbius and conformal maps; graphics hooks | 600 plus 800 in graphics |
| 6 | Complex gamma and zeta | 700 |

Phases 1 and 2 are independent and each useful alone. Phase 3 is the hard one.

## 4. A number theory package: `@ratmath/numtheory`

### 4.1 What exists already

In algebraic: `factorInteger`, `crt`, `modPow`, `modInverse`, `jacobi`,
`sqrtMod`, `primitiveRoot`, `multiplicativeOrder`, `discreteLogMod`, `totient`
(`src/nt.js`, 208 lines); continued fractions, convergents, Pell and
fundamental units of real quadratic fields (`src/cf.js`, 261 lines);
`isProbablePrime`, prime and Galois fields; LLL and `recognize`; number fields
and factoring over them. Core has Stern–Brocot and Farey mediants. Analytic has
`bernoulli` and the gamma family.

### 4.2 Scope

**Elementary (a first course).**
- Primes: sieves, next and previous prime, counting, and primality *proofs*
  with a checkable certificate (Pocklington, then elliptic curve certificates),
  which suits the house rule that an answer is either proven or says it is not.
- Factoring beyond trial division and rho: p−1, elliptic curve method, a small
  quadratic sieve.
- Arithmetic functions (σ, τ, μ, φ, Carmichael λ), Dirichlet convolution,
  perfect and amicable numbers, partitions.
- Congruences: linear systems, polynomial congruences, Hensel lifting, indices.
- Quadratic residues and reciprocity with the steps shown; Kronecker symbol.
- Sums of two, three and four squares; Pythagorean triples.
- Diophantine equations: linear, general Pell, norm equations, Frobenius coin
  problem, Egyptian fractions.
- Binary quadratic forms: reduction, classes, the class number h(D),
  representation of integers.
- Teaching cryptography: RSA, Diffie–Hellman, ElGamal, elliptic curve versions.

**p-adic numbers.** Z_p and Q_p with tracked precision as a `Refinement`
(more digits on request), valuations, Hensel, p-adic expansions of rationals
(which connects to Core's repeating expansions in any base), `exp` and `log`.

**Elliptic curves.** Group law over Q and F_p, point counting (naive, then
Schoof), torsion over Q (Nagell–Lutz, Mazur's list), heights, reduction types.
Ranks in general are out of scope. Supplies the elliptic curve method and
primality certificates above.

**Algebraic number theory.** Integral bases and discriminants (Round 2),
prime ideal factorisation (Dedekind–Kummer), class groups and unit groups,
with quadratic fields done completely first through forms. This is where the
three algebraic leftovers from section 2 land.

**Analytic number theory.** Exact π(x) (Meissel–Lehmer), Dirichlet characters,
certified values of ζ and L-functions on the real line (analytic), and on the
plane once the complex package has phase 6.

### 4.3 Where the existing functions live

The rule (James, 2026-11-25): a function lives as high up the dependency
chain as is reasonable, that is, in the most specialised package that it
belongs to. It stays lower only when a lower package needs it for its own
work, and then the higher package re-exports it, so that users find the whole
subject in one place and no dependency loop appears.

Applied here, with `@ratmath/numtheory` depending on algebraic:

- **Stays in algebraic, re-exported by numtheory:** the modular kernel that
  algebraic's finite fields import today (`multiplicativeOrder`,
  `primitiveRoot`, `discreteLogMod`, `sqrtMod`, `isProbablePrime`) and what
  those rest on (`modPow`, `modInverse`, `factorInteger`).
- **Moves up to numtheory:** continued fractions, convergents, Pell and
  fundamental units (`src/cf.js`), which nothing else in algebraic imports,
  and the remaining functions of `src/nt.js` that algebraic does not use
  itself (`crt`, `jacobi`, `totient` are the candidates; the exact split is
  settled when phase 1 is written). Algebraic cannot re-export these without
  a loop, so their import path changes. Neither package is on npm yet, so
  that costs little now.
- **Stays in algebraic:** LLL and `recognize`, which serve algebraic numbers
  first; numtheory re-exports LLL.
- Number fields stay in algebraic; integral bases, class groups and units are
  new and are written in numtheory.

Depends on core and algebraic; analytic for the certified real values; complex
only for phase 6.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Primes with certificates, arithmetic functions, congruences, reciprocity, sums of squares, re-exports | 1,400 |
| 2 | Diophantine equations and binary quadratic forms | 900 |
| 3 | p-adic numbers | 500 |
| 4 | Elliptic curves; ECM and curve-based certificates | 1,400 |
| 5 | Integral bases, discriminants, prime ideal factorisation | 900 |
| 6 | Class groups and unit groups | 1,500 |
| 7 | π(x), characters, ζ and L values | 800 |

## 5. An abstract algebra package: `@ratmath/structures`

Named for the algebraic structures it holds (groups, rings, fields, modules),
which also keeps it apart from `algebraic`.

### 5.1 What exists already

Algebraic's `PermGroup` (351 lines): membership, orbits, cosets, normality,
conjugates, derived series, solvability, all subgroups and their classes for
small groups, `symmetric`, `alternating`. Galois groups with the full
correspondence. Finite fields, number fields, splitting fields, one-variable
polynomial arithmetic and factoring. Linalg has Smith and Hermite normal forms
and the rational canonical and Jordan forms.

### 5.2 Scope

**Groups.**
- Named families (cyclic, dihedral, quaternion, symmetric, alternating, direct
  and semidirect products, matrix groups over finite fields) and a table of
  small groups.
- Cayley tables, element orders, centre, centraliser, normaliser, conjugacy
  classes, commutators, composition series; nilpotent and solvable tests.
- Subgroup lattice with pictures (graphics); quotients; Sylow subgroups.
- Homomorphisms, kernels, images, the isomorphism theorems on examples,
  isomorphism testing for small groups, automorphism groups.
- Actions: orbits, stabilisers, and the orbit-counting lemma. Pólya counting
  itself is in the discrete package (section 7), built on this.
- Finitely presented groups by coset enumeration (Todd–Coxeter).
- Finitely generated abelian groups through the Smith normal form in linalg.
- Schreier–Sims so that permutation groups of large degree work; this also
  lifts the degree limit on Galois groups.
- Character tables of small groups (a stretch item).

**Rings.**
- Z/n, Gaussian and Eisenstein integers, quadratic integer rings, with
  Euclidean division where it exists and the standard examples of unique
  factorisation failing.
- Polynomials in several variables over Q, finite fields and number fields.
  This one type also unlocks linalg's symbolic parameters and exact algebraic
  curves of degree three and up in graphics.
- Ideals, quotient rings, the Chinese remainder theorem for rings, prime and
  maximal tests in the computable cases.
- Gröbner bases with a chosen monomial order and replayable reductions
  (`WORK_PLAN.md` D5 asks for exactly that), elimination, solving polynomial
  systems exactly with algebraic numbers as answers.

**Fields.** Towers of extensions, degrees, minimal polynomials of combinations
(algebraic has the machinery), constructibility with ruler and compass,
solvability by radicals with the radical expression when the group is
solvable, Conway polynomials for standard finite fields.

**Modules.** Over Z and over Q[x]: the structure theorem computed, with the
link to the rational canonical form made explicit.

### 5.3 Dependencies and phases

Depends on core, algebraic (fields, factoring, `PermGroup`) and linalg (normal
forms). By the same rule as in 4.3, `PermGroup` stays in algebraic because
Galois groups are computed with it, and this package re-exports and extends
it. New group code, including Schreier–Sims, is written here; if Galois groups
of higher degree later need Schreier–Sims, only that kernel moves down.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Named groups, tables, classes, quotients, Sylow, homomorphisms, actions and counting | 1,800 |
| 2 | Several-variable polynomials | 900 |
| 3 | Rings, ideals, quotients, the quadratic integer rings | 1,000 |
| 4 | Gröbner bases, elimination, polynomial systems | 1,200 |
| 5 | Presentations and coset enumeration; abelian groups; modules | 1,000 |
| 6 | Schreier–Sims; field towers, radicals, constructibility; Conway polynomials | 1,300 |
| 7 | Character tables | 900 |

Phase 2 is worth doing early whichever package order is chosen, because
linalg and graphics both wait on it.

## 6. A probability and statistics package: `@ratmath/stats`

Added on review (James, 2026-11-25): probability and statistics become one
Shelf library, with the ability to generate random data from a model that is
known, disturb it with noise, and see whether tests and regressions recover
what was put in.

### 6.1 Three kinds of answer

The other libraries give answers that are exact or certified. This one has a
third kind, and it must never be confused with the other two:

- **Exact.** Rational data give rational statistics: means, variances,
  least-squares coefficients, probabilities on finite spaces, and the p-values
  of exact tests (binomial, Fisher, permutation).
- **Certified.** Values of continuous distributions (density, cumulative
  probability, quantile) are enclosures, so a p-value is an interval and the
  comparison with a significance level is either decided or reported as too
  close to call at this precision.
- **Simulated.** The result of a random experiment. It is labelled as such,
  carries its seed so it can be repeated, and reports its own sampling error
  (an exact binomial interval for a rate, for instance).

### 6.2 Scope

**Probability.**
- Finite sample spaces, events, conditional probability, independence, Bayes,
  random variables, joint and marginal tables, all exact.
- Discrete distributions (binomial, geometric, negative binomial,
  hypergeometric, Poisson, uniform) with exact probabilities.
- Continuous distributions (normal, t, chi-square, F, exponential, gamma,
  beta, uniform, lognormal, Weibull, Cauchy) with certified density,
  cumulative probability and quantile. This needs the incomplete gamma and
  beta functions, which belong in analytic next to the gamma family (about
  500 lines there).
- Expectation, variance, moments and generating functions; sums of
  independent variables (exact convolution for discrete ones); transformations;
  order statistics.
- The law of large numbers and the central limit theorem as experiments, with
  the certified limit drawn beside the simulated histogram.
- Markov chains through linalg: stationary distributions, absorption
  probabilities and expected hitting times, exact.

**Statistics.**
- Descriptive statistics, exact, including data given as intervals
  (measurements with stated uncertainty), as the RiX plugin has.
- Estimation: method of moments, maximum likelihood for the standard families,
  confidence intervals.
- Tests: z and t (one sample, paired, two samples), proportions, chi-square
  (fit and independence), F and one-way analysis of variance, correlation;
  exact binomial and Fisher tests; sign, Wilcoxon, Mann–Whitney,
  Kolmogorov–Smirnov; permutation tests; the bootstrap.
- Regression: simple, multiple and polynomial least squares (exact
  coefficients for rational data, through linalg), inference on coefficients,
  residuals and diagnostics, prediction bands. Logistic regression later.
- Bayesian updating for the conjugate families.

**Generating data, and testing the tests.** This is the part James asked for
by name.

- A seeded generator that gives the same stream everywhere (JavaScript and the
  Rust kernels), so every simulated result can be reproduced from its seed.
  The discrete package needs it too (random permutations and graphs), so by
  the rule of 4.3 it lives there and is re-exported here.
- Samples from any distribution above. Two speeds, in the house manner: floats
  for exploring, and exact rational samples obtained by pushing an exact
  uniform draw through a certified quantile.
- A *model* is a known truth plus a description of how observations depart
  from it: for example `y = 2 + 3x` with normal noise of a given spread.
  Departures can be combined:
  - additive noise from any distribution;
  - noise whose spread depends on x;
  - contamination: with a stated probability an observation is drawn from a
    second distribution instead (heavy tails, outliers);
  - noise in the x values as well as in y;
  - rounding, censoring and missing values.
- An *experiment* draws many data sets from a model, runs a procedure on each
  and compares the outcome with the truth that generated them:
  - how often a 95% interval really contains the true value;
  - how often a test rejects when the null hypothesis is true, and its power
    as the true effect grows;
  - bias and spread of an estimator;
  - how often a regression or a model-selection rule recovers the true form,
    and how that degrades as noise or contamination is turned up;
  - what happens when a test's assumptions are deliberately broken.
- Every experiment's result is a *simulated* answer in the sense of 6.1.

**Pictures** in graphics: histograms, box plots, scatter plots with fitted
lines and bands, quantile plots, density curves with certified shaded areas,
power curves, and the animated build-up of a sampling distribution.

### 6.3 Relation to RiX, dependencies and phases

RiX has `probability` and `stats` plugins. As decided for the other packages
(3.2), they stay independent and this library is added to `ratmath/rix-bridge`.

Depends on core, analytic (special functions, integrals), linalg (regression,
Markov chains) and the discrete package of section 7 (counting and the
enumeration of finite sample spaces, so that discrete probability is not
written twice). Nothing here depends on the complex, number theory or
structures packages.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Finite probability, discrete distributions, descriptive statistics, exact tests | 1,100 |
| 2 | Continuous distributions, certified | 1,200, plus 500 in analytic |
| 3 | Seeded generation, models with noise and contamination, experiments | 1,300 |
| 4 | Estimation, confidence intervals, the standard tests | 1,000 |
| 5 | Regression and analysis of variance | 1,100 |
| 6 | Rank tests, permutation tests, the bootstrap | 700 |
| 7 | Markov chains; conjugate Bayesian updating | 700 |
| 8 | Graphics hooks | 800 in graphics |

Phases 1 to 3 already make the exploration James describes possible for the
exact tests; phases 4 and 5 bring in the tests and regressions most courses
use.

## 7. A discrete mathematics package: `@ratmath/discrete`

Added on review (James, 2026-11-25): combinatorics, graph theory, game theory
and related topics in one library, which the statistics package pulls in for
discrete probability.

### 7.1 Scope

**Counting and enumeration.**
- Permutations, combinations, multisets, compositions, partitions of integers
  and of sets; binomial and multinomial coefficients; Stirling, Bell, Catalan
  and Fibonacci numbers, derangements. All exact integers of any size.
- Lazy enumeration of each family with ranking and unranking, so the millionth
  permutation is found without listing the others, and a uniformly random one
  can be drawn.
- Inclusion and exclusion; the pigeonhole principle on examples.
- Pólya counting of colourings up to symmetry, with groups from algebraic's
  `PermGroup`.

**Generating functions and recurrences.**
- Ordinary and exponential generating functions as exact power series, with
  rational ones kept in closed form (one-variable rational functions from
  algebraic).
- Linear recurrences with constant coefficients solved in closed form, the
  characteristic roots being exact algebraic numbers (so Fibonacci comes out
  with its √5), and evaluated at huge indices by matrix powers.
- Sums in closed form for the standard cases; divide-and-conquer recurrences.

**Graphs.**
- Graphs, directed graphs, multigraphs, weights as exact rationals; the
  standard families; subgraphs, complements, products, line graphs.
- Traversal, connectivity, components, bridges and cut vertices, strongly
  connected components, topological order.
- Shortest paths, minimum spanning trees, Euler tours, Hamilton cycles (by
  search, for small graphs), trees and Prüfer codes.
- Matchings (bipartite with Hall's condition as the certificate of failure,
  then general), flows with the minimum cut as certificate, stable matching.
- Colouring, the chromatic polynomial, planarity with a Kuratowski subgraph
  when it fails, independent sets and cliques for small graphs.
- Isomorphism and automorphism groups of small graphs (returned as a
  `PermGroup`).
- Algebraic graph theory through linalg: adjacency and Laplacian spectra with
  exact eigenvalues, the number of spanning trees from the matrix-tree
  theorem, counts of walks.
- Random graphs and random permutations from the seeded generator.

Every "no" comes with its reason where one exists (a cut, a Hall violator, a
Kuratowski subgraph), in keeping with the libraries' habit of proof.

**Games.**
- Two-player zero-sum matrix games solved exactly: optimal mixed strategies
  and the value as rationals, by exact linear programming.
- Bimatrix games: dominance, pure and mixed Nash equilibria (support
  enumeration, then Lemke–Howson), all exact.
- Games in extensive form and backward induction.
- Combinatorial games: Nim, Sprague–Grundy values, sums of games, and the
  values of small partizan games.
- Cooperative games: the Shapley value and the core.
- Voting and apportionment methods, fair division, where exact fractions
  matter to the outcome.

**Exact linear programming.** A rational simplex method returning the optimum
together with the dual solution that proves it, and integer programs by branch
and bound for small cases. The games need it; it is also most of what the
optimisation row in section 8 lacked. By the rule of 4.3 it starts here, and
moves down to linalg only if linalg comes to need it.

**Sets, relations and Boolean functions.** Relations and their properties,
equivalence classes, partial orders with Hasse diagrams, lattices; Boolean
functions, truth tables and normal forms. Proof systems stay in RiX's `logic`
plugin.

**Codes and designs** (later). Linear codes over algebraic's finite fields
(Hamming, Reed–Solomon), error correction shown step by step; Latin squares
and block designs.

**Pictures** in graphics: graph layouts (circular, layered, force-directed,
planar where the graph is planar), highlighted paths, cuts and colourings,
game trees, Hasse diagrams and subgroup lattices (shared with section 5).

### 7.2 Relation to RiX, dependencies and phases

RiX has small `combinatorics` and `graph` plugins. As with the other packages
they stay independent, and this library is added to `ratmath/rix-bridge`.

Depends on core, algebraic (`PermGroup`, algebraic numbers, rational
functions, finite fields) and linalg (spectra, determinants, matrix powers).
The statistics package depends on this one, not the other way round.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Counting, enumeration with ranking, the seeded generator | 1,000 |
| 2 | Graphs: structure, traversal, paths, trees, families | 1,300 |
| 3 | Generating functions and recurrences | 900 |
| 4 | Matchings, flows, colouring, planarity, isomorphism, spectra | 1,600 |
| 5 | Exact linear programming; matrix and bimatrix games | 1,100 |
| 6 | Combinatorial, extensive-form and cooperative games; voting | 1,000 |
| 7 | Sets, relations, partial orders, Boolean functions; Pólya counting | 700 |
| 8 | Codes and designs | 700 |
| 9 | Graph drawing | 900 in graphics |

## 8. Coverage of an undergraduate curriculum

"Covered" here means: the calculations a course asks a student to carry out
can be done in the JavaScript libraries with an exact or certified answer, and
most have a picture or a worked form. It does not mean proofs, which are the
point of the upper-level courses and which no computing library supplies. The
percentages are a judgement from each course's usual list of topics, not a
measurement. The RiX column records that a plugin exists; those were assessed
from their READMEs only.

| Course | Libraries today | RiX plugins today | After the five packages | Still missing |
|---|---|---|---|---|
| Calculus I and II | 90% | yes | 90% | Symbolic antiderivatives and series convergence tests as named steps; limits needing cancellation |
| Calculus III (several variables, vector calculus) | 80% | part | 80% | Green, Stokes and divergence theorems as checked identities; change of variables shown as a step |
| Linear algebra | 95% | yes | 95% | Symbolic parameters (section 2); abstract vector spaces and inner products beyond R^n |
| Differential equations (ordinary) | 70% | part | 75% | Symbolic solution methods, Laplace transforms, systems and phase portraits as features, series solutions at singular points, stability |
| Complex analysis | 10% | 15% | 85% | Riemann mapping in general, analytic continuation beyond named paths, harmonic functions and Dirichlet problems |
| Number theory | 40% | little | 90% | Ranks of elliptic curves, the deeper analytic theory |
| Abstract algebra I (groups) | 30% | none | 85% | Infinite groups beyond presentations, representation theory beyond character tables |
| Abstract algebra II (rings, fields, Galois) | 60% | part | 90% | Galois groups of high degree, commutative algebra beyond Gröbner bases, modules over general rings |
| Real analysis | 30% | part | 30% | See below |
| Numerical analysis | 50% | part | 50% | Interpolation and splines, quadrature rules as objects, iterative solvers, floating-point error analysis as a subject |
| Probability | 0% | yes | 85% | Measure-theoretic foundations; stochastic processes beyond Markov chains |
| Statistics | 0% | yes | 80% | Generalised linear models beyond logistic, time series, multivariate methods, experimental design |
| Discrete mathematics and combinatorics | 10% | part | 85% | Asymptotic and analytic combinatorics; proof techniques as such |
| Graph theory | 0% | part | 80% | Hard problems beyond small graphs; extremal and random graph theory as theory |
| Game theory | 0% | none found | 75% | Games of incomplete information, repeated and evolutionary games, mechanism design |
| Geometry | 50% | yes | 55% | Transformations as groups, non-Euclidean models, projective geometry; the editor |
| Logic and set theory | 0% | part | 20% | Relations and Boolean functions are in the discrete package; proof systems and first-order logic stay in RiX |
| Topology | 0% | none | 5% | See below |
| Differential geometry | 10% | none | 10% | Curvature and torsion, first and second fundamental forms, geodesics |
| Partial differential equations | 10% | none | 10% | Separation of variables with Fourier series is there; heat, wave and Laplace solvers are not |
| Optimisation | 20% | yes | 55% | Nonlinear constrained optimisation, Lagrange multipliers as a named method, convex optimisation |

Read as a whole: the calculus sequence, linear algebra and a good part of
differential equations are covered now. Those are the courses every
mathematics, science and engineering student takes. Of the upper-level core of
a mathematics major (real analysis, abstract algebra, complex analysis, with
number theory as the most common elective), the libraries cover parts of
algebra and number theory only. The first three packages would bring complex
analysis, number theory and both algebra courses to roughly the level the
calculus sequence is at today. That closes three of the four core upper-level
subjects. The statistics package adds the two courses that, after calculus,
the largest number of students outside mathematics take, and the discrete
package adds the course that computer science students take in their place.

## 9. What would remain

In rough order of how well each suits exact and certified computation:

1. **Differential equations, second half.** Laplace transforms, linear systems
   with phase portraits (linalg's matrix exponential plus graphics), stability,
   and the three classical partial differential equations by separation of
   variables. Extends analytic and graphics, no new package.
2. **Numerical analysis.** Mostly a matter of exposing as teaching objects
   what the libraries do internally (interpolation, quadrature, Newton with a
   proof, error bounds). The certified answer next to the classical method's
   answer is a good lesson in itself.
3. **Differential geometry of curves and surfaces.** Curvature, torsion,
   fundamental forms, geodesics as ODEs. Sits naturally on analytic's fields
   and graphics' 3D. About 1,500 lines.
4. **Optimisation beyond linear programming.** Lagrange multipliers and
   constrained nonlinear problems. RiX has an `optimize` plugin.
5. **Real analysis and topology.** These are proof courses. What computing can
   offer real analysis is already the project's own subject (reals as families
   of rational intervals, sequences with certified limits, uniform against
   pointwise convergence shown with enclosures). Point-set topology offers
   little to compute; finite simplicial homology over Z is computable through
   Smith normal form and would be a small addition (about 600 lines) rather
   than coverage of the course.
6. **Logic and set theory** beyond relations and Boolean functions. Leave to
   RiX's `logic` plugin.
7. **The geometry editor**, which is an app.

## 10. Order, and decisions made

Suggested order, with the reason for each step:

1. Several-variable polynomials (section 5, phase 2), because linalg's
   symbolic parameters and graphics' higher curves wait on them.
2. Complex phases 1 and 2 (numbers, branches, quaternions and octonions).
   Self-contained, and they settle the exact complex type that linalg's
   complex entries need.
3. Discrete phase 1 (counting, enumeration, the seeded generator), because
   statistics builds on it.
4. Statistics phases 1 to 3. The most widely used subject, and enough for the
   first experiments with generated data.
5. Discrete phase 2 (graphs) and number theory phases 1 to 3. Both mostly
   independent of everything else.
6. Abstract algebra phases 1 and 3 (groups and rings).
7. Statistics phases 4 and 5 (the standard tests and regression) and discrete
   phase 5 (linear programming and matrix games).
8. Complex phases 3 and 4 (the analysis proper). The hardest single piece.
9. The remaining phases as interest directs.

Decisions so far. The packages are named `@ratmath/complex`, `@ratmath/numtheory` and
`@ratmath/structures` (James, 2026-11-25).
The hypercomplex numbers stay in the complex package, as the subpath
`@ratmath/complex/cayley` (James, 2026-11-25).
The RiX plugins stay independent of the Shelf for now; `ratmath/rix-bridge`
is to be widened so each new package is reachable from RiX (3.2).
A function lives in the most specialised package it belongs to, unless a
lower package needs it, in which case the higher one re-exports it (4.3, 5.3).

Probability and statistics become one library, `@ratmath/stats`, with
generation of random data and noise for testing the tests (section 6; James,
2026-11-25).

Discrete mathematics (combinatorics, graph theory, game theory) becomes a
library, `@ratmath/discrete`, which the statistics package uses for discrete
probability (section 7; James, 2026-11-25).

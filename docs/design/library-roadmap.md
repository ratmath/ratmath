# Library roadmap: what is left, and ten further packages (design)

Status: proposal, written 2026-11-25. Nothing in sections 3 to 13 is built.
It follows the gold-plating pass of 2026-11-25 (linalg#4, analytic#9,
algebraic#5, graphics#7, all merged) and answers four questions:

1. What from the gold-plating document was not done?
2. What would a complex package, a number theory package and an abstract
   algebra package contain? (A statistics package and a discrete mathematics
   package, and packages for logic, symbolic algebra, dynamical systems,
   geometry and finance, were added to the plan on review (sections 6 to 12),
   with extensions of the current libraries (section 13).)
3. How much of an undergraduate mathematics curriculum do the libraries cover
   today, and how much would those packages add?
4. What would still remain?

Line counts for future work are estimates made from the size of comparable
code already in the libraries. They are for ordering the work, not promises.
Curriculum percentages in section 14 are a judgement, explained there.

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
| `@ratmath/structures` | 1,900 | First part only: polynomials in several variables over any exact field, division, gcd, factoring over Q, resultants and discriminants, rational functions, symmetric polynomials |
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
oversees them. The ten packages below would join the Shelf there as they
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
| Features of parametric curves (self-intersections, cusps, arc length) | 500 | Graphics, with the differential geometry of section 13.2 |
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
| 1 | Exact complex rationals, rectangles and discs, polar form, elementary functions with branch reporting (built 2026-11-26) | 1,400 |
| 2 | Cayley–Dickson numbers, quaternion rotations, integer quaternions (built 2026-11-26) | 700 |
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
  **Built** (2026-11-25) as the first part of
  [`@ratmath/structures`](https://github.com/ratmath/structures), with
  division, gcd, factoring over Q, resultants, rational functions and
  symmetric polynomials; it is listed on the
  [Shelf page](https://ratmath.com/shelf/#structures). Its field of rational
  functions already works with linalg's field module for symbolic parameters;
  exact algebraic curves of degree three and up in graphics can now build on
  it.
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
| 2 | Several-variable polynomials (built 2026-11-25) | 900 |
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
optimisation row in section 14 lacked. By the rule of 4.3 it starts here, and
moves down to linalg only if linalg comes to need it.

**Sets and relations.** Relations and their properties, equivalence classes,
partial orders with Hasse diagrams, lattices. Boolean functions, truth tables
and normal forms come from the logic package (section 8) and are re-exported
here, since a discrete mathematics course expects to find them.

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
functions, finite fields), linalg (spectra, determinants, matrix powers) and
logic (Boolean functions).
The statistics package depends on this one, not the other way round.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Counting, enumeration with ranking, the seeded generator (built 2026-11-26) | 1,000 |
| 2 | Graphs: structure, traversal, paths, trees, families | 1,300 |
| 3 | Generating functions and recurrences | 900 |
| 4 | Matchings, flows, colouring, planarity, isomorphism, spectra | 1,600 |
| 5 | Exact linear programming; matrix and bimatrix games | 1,100 |
| 6 | Combinatorial, extensive-form and cooperative games; voting | 1,000 |
| 7 | Sets, relations, partial orders; Pólya counting | 600 |
| 8 | Codes and designs | 700 |
| 9 | Graph drawing | 900 in graphics |

## 8. A logic package: `@ratmath/logic`

Added on review (James, 2026-11-25). The calculator is JavaScript, so an
engine for exploring formal logic has to exist in JavaScript; as a package it
gets its own tests, and the app holds only entry and display.

### 8.1 Why there is something to compute

An undergraduate logic course is full of questions with a definite answer that
a machine can give and justify: is this formula valid, and if not, what is a
countermodel; are these two equivalent; does this conclusion follow; is this
derivation correct, and if not, which line fails and why; is there a
derivation at all. RiX's `logic` plugin (about 750 lines) already answers these
for propositional logic, with truth tables to 16 atoms, normal forms, a
natural deduction checker with subproofs, tableaux and a sequent calculus. It
has no quantifiers.

The house rule carries over directly. Every answer is one of three: **valid**
with a proof that can be replayed, **invalid** with a countermodel that is
checked before it is returned, or **unresolved** within the stated budget.
"No countermodel with up to five elements" is reported as exactly that, never
as validity.

### 8.2 Scope

**Propositional logic.**
- Formulas from text in the common notations; syntax trees.
- Truth tables; validity, satisfiability, equivalence and consequence, by a
  satisfiability solver that returns a model or a refutation.
- Normal forms; simplification (Quine–McCluskey, Karnaugh maps); circuits of
  gates and their equivalence.

**Proofs.**
- Checkers for natural deduction in the Fitch style, sequent calculus,
  tableaux (truth trees) and resolution. A failed check names the line, the
  rule and what was missing.
- Rule sets are data, because textbooks differ in their rules and their names.
  The classical and intuitionistic systems are kept apart.
- Proof search that returns a proof the checker accepts, with hints (the next
  line only) for a student who is stuck.

**Predicate logic.**
- Quantifiers, equality, function symbols; free and bound variables,
  substitution, prenex and Skolem forms.
- Finite structures: evaluating a sentence in a given model, with the steps;
  searching for a model or a countermodel up to a stated size.
- Proof checking with the quantifier rules; tableaux with a budget.
- Syllogisms and Venn diagrams.

**Atoms that are arithmetic facts.** A proposition may be an exact comparison
of numbers or intervals from Core, decided or left undecided, as RiX's
`ExactProposition` does. This connects the package to the rest of the Shelf.

**Modal logic.** Kripke models, truth at a world, validity on finite frames,
tableaux for the systems K, T, S4 and S5.

**Sets and infinity.** Ordinals below ε₀ in Cantor normal form with their
arithmetic and comparison; the rules of cardinal arithmetic; Peano arithmetic
by structural recursion; Gödel numbering shown on real formulas.

**Computability.** Finite automata and regular expressions (conversion,
minimisation, equivalence with a distinguishing word when they differ),
grammars and pushdown automata, Turing machines with a step budget,
primitive recursive functions. This is the theory of computation course that
often accompanies logic.

**Pictures** in graphics: proof trees, truth trees, Venn diagrams, Kripke
frames, automata. The last two share graph drawing with the discrete package.

### 8.3 Relation to RiX, dependencies and phases

RiX's `logic` plugin stays independent, as decided in 3.2, and this library is
added to `ratmath/rix-bridge`.

Depends on core only. The discrete package depends on it for Boolean functions.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Propositional formulas, tables, normal forms, satisfiability, simplification (built 2026-11-26) | 900 |
| 2 | Proof checkers, rule sets, proof search and hints (built 2026-11-26) | 1,200 |
| 3 | Predicate logic: syntax, finite models, model search, quantifier rules | 1,100 |
| 4 | Circuits, Karnaugh maps, syllogisms | 400 |
| 5 | Modal logic | 500 |
| 6 | Ordinals, cardinals, Peano arithmetic, Gödel numbering | 500 |
| 7 | Automata, grammars, Turing machines | 1,000 |
| 8 | Pictures | 500 in graphics |

## 9. A symbolic algebra package: `@ratmath/symbolic`

Added on review (James, 2026-11-25). The Shelf computes with numbers,
polynomials and certified functions but cannot yet manipulate expressions in
general. That is most of school and college algebra and a visible part of
calculus.

### 9.1 What makes this one different

A computer algebra system is usually trusted on faith. Here every rewriting
step can be checked, because the certified libraries sit underneath.

- **Equality is a three-way answer.** Two expressions are *proven equal* (they
  reduce to the same normal form, as polynomials and rational functions do),
  *proven different* (a certified evaluation at some point separates them), or
  *undecided*.
- **Rules know their domains.** √(x²) is |x|, not x; log(ab) = log a + log b
  needs a and b positive. A rule is applied only when its condition is proven
  from the stated assumptions, and otherwise the condition is returned with
  the result.
- **Results are verified before they are returned.** An antiderivative is
  differentiated and compared with the integrand; a solution is substituted
  back; a root that the algebra introduced (squaring both sides) is caught.

### 9.2 Scope

- **Expressions.** The same expression tree that analytic and graphics already
  share, so anything passes between the three. Substitution, pattern matching,
  user rules, assumptions on variables (positive, integer, in an interval,
  stated as Core regions).
- **Rewriting.** Expand, collect, factor (algebraic, and several variables
  from the structures package), a normal form for rational expressions,
  partial fractions, rules for powers, roots, logarithms and trigonometric
  functions, simplification of nested radicals (algebraic has denesting).
- **Solving in one variable.** Linear, polynomial (exact roots as algebraic
  numbers), rational, radical, exponential, logarithmic and trigonometric
  equations with their whole families of solutions; absolute values;
  inequalities, whose answers are Core's interval sets. When no formula
  exists, the certified roots from analytic are returned and said to be
  numeric.
- **Systems.** Linear through linalg, polynomial through elimination in the
  structures package.
- **Calculus in closed form.** Antiderivatives by the methods of the course
  (substitution, parts, partial fractions, trigonometric substitution), all
  rational functions completely, each verified. Limits with the cancellation
  that analytic's growth calculus refuses today. Taylor and Laurent
  expansions. Sums in closed form for the standard families.
- **Steps.** Every transformation is a named rule, and a result carries the
  list of steps that produced it, for display by an app. A step is shown only
  if it was checked.

The general decision procedures (Risch integration and the like) are out of
scope; the methods are those a student is taught.

### 9.3 Relation to RiX, dependencies and phases

RiX has `cas`, `symbolic` and `calculus` plugins. They stay independent, and
this library is added to `ratmath/rix-bridge`.

Depends on core, analytic (the expression tree, derivatives, enclosures) and
algebraic (factoring, algebraic numbers); the structures package for several
variables. Sections 6 (maximum likelihood in closed form) and 13.1 (solution
formulas for differential equations) use it where it exists and work without
it otherwise.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Expressions, assumptions, rewriting, three-way equality (built 2026-11-26) | 1,500 |
| 2 | Solving equations and inequalities in one variable | 1,300 |
| 3 | Antiderivatives with verification; limits; expansions | 1,600 |
| 4 | Systems, trigonometric families, sums | 900 |
| 5 | Step records for every operation | 700 |

## 10. A dynamical systems package: `@ratmath/chaos`

Added on review (James, 2026-11-25).

### 10.1 Why exact arithmetic matters here

Chaos is where floating point fails in plain sight: after about fifty steps of
the logistic map a double has no correct digits left, and nothing warns you.
An exact rational orbit is the true orbit, and an interval orbit says honestly
how much is still known. Showing the float orbit beside the true one, and the
step at which they part, is the first lesson of the subject and one that only
exact arithmetic can give.

Results follow the three kinds of section 6.1: exact (a periodic point as an
algebraic number), certified (an enclosure of an orbit or of a Lyapunov
exponent), and estimated (a picture built from finitely many iterates, said
to be so).

### 10.2 Scope

**Maps of the line.**
- Orbits, exact and enclosed; fixed and periodic points of polynomial maps as
  exact algebraic numbers, with their multipliers and stability.
- Cobweb diagrams; bifurcation diagrams with the periodic windows located and
  proven; period doubling and an enclosure of Feigenbaum's constant.
- Lyapunov exponents; sensitive dependence measured exactly.
- Symbolic dynamics: itineraries, kneading sequences, Sharkovskii's order.

**Maps of the plane.** Hénon, the standard map, the cat map (whose rational
points have exact periodic orbits), the horseshoe; fixed points and their
stable and unstable directions from linalg.

**Flows.** Lorenz, Rössler, the pendulum, Van der Pol, through analytic's
validated solver. Poincaré sections; a periodic orbit proven to exist from an
enclosure of the return map; equilibria and the bifurcations they undergo
(saddle-node, pitchfork, Hopf) located as a parameter moves. Phase portraits
are shared with section 13.1.

**Fractals.**
- Iterated function systems with exact affine maps (Sierpinski, Koch,
  Barnsley's fern); Cantor sets through Core's expansions in any base;
  similarity dimension exactly and box-counting dimension as an estimate.
- Julia sets and the Mandelbrot set with each pixel classed as inside, outside
  or undecided, from the complex package. Newton's method basins.
- L-systems.

**Cellular automata.** The elementary rules and Life, as the discrete end of
the subject.

**Pictures** in graphics: cobwebs, bifurcation diagrams, attractors in two and
three dimensions, return maps, fractals with honest pixels, all animated in a
parameter.

### 10.3 Relation to RiX, dependencies and phases

RiX has a `fractals` plugin; it stays independent, and this library is added
to `ratmath/rix-bridge`.

Depends on core, analytic (enclosures, the validated solver), algebraic
(periodic points) and linalg (linearisation); the complex package for phase 5.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Maps of the line: orbits, periodic points, stability | 900 |
| 2 | Bifurcation diagrams, Lyapunov exponents, symbolic dynamics | 1,000 |
| 3 | Maps of the plane; flows, sections, periodic orbits, bifurcations | 1,300 |
| 4 | Iterated function systems, Cantor sets, dimensions, L-systems, cellular automata | 900 |
| 5 | Julia and Mandelbrot sets, Newton basins | 600 |
| 6 | Pictures | 1,000 in graphics |

## 11. A geometry package: `@ratmath/geo`

Added on review (James, 2026-11-25): a library for exploring different
geometries, continuous and discrete, relying on the other packages as needed.

### 11.1 What exists already

Graphics has exact plane geometry: points, lines, segments, rays, polygons and
circles with coordinates in Q or Q(√d), intersections, reflections,
bisectors, circumcircles, convex hulls, and exact regions bounded by lines and
conics. By the rule of 4.3 that kernel stays in graphics, which needs it, and
this package re-exports and extends it.

### 11.2 Scope

**The Euclidean plane and space.**
- Ruler and compass constructions with exact coordinates for every
  constructible point (algebraic's numbers lift graphics' limit of a single
  square root); the classical triangle centres, circles and lines.
- Theorems checked on exact configurations, and proved in general by
  elimination (the structures package's Gröbner bases): the hypotheses and the
  conclusion become polynomials, and the proof is a certificate that can be
  replayed.
- Loci, inversion in a circle, conics in every form.
- Solids: polyhedra with exact coordinates in number fields (the regular and
  semi-regular ones), duals, Euler's formula, volumes and angles.

**Transformations and symmetry.** Isometries of the plane and of space
classified from their matrices; similarities and affine maps; symmetry groups
of figures as `PermGroup`s; the frieze groups and the seventeen wallpaper
groups; tilings.

**Other continuous geometries.**
- Spherical: great circles, triangles and their angle excess, with exact
  trigonometric values where they exist and enclosures otherwise.
- Hyperbolic: the disc, half-plane, Klein and hyperboloid models and the maps
  between them; geodesics, distance, angle, area as angle defect; isometries
  as Möbius maps from the complex package; regular tilings.
- Projective: homogeneous coordinates, duality, cross-ratio, projective maps,
  the theorems of Desargues and Pappus, conics.
- The parallel postulate examined in each model, with logic's finite
  structures for the small axiomatic examples.

**Discrete geometries.**
- Finite affine and projective planes over algebraic's finite fields (the
  Fano plane and its relatives), incidence structures, and their link to the
  designs of the discrete package.
- Lattice geometry: Pick's theorem, lattice polygons, visible points.
- Computational geometry with exact predicates: convex hulls, triangulations,
  Delaunay and Voronoi diagrams, point location, polygon operations. With
  rational arithmetic the degenerate cases that defeat floating-point
  implementations cannot go wrong.
- Other metrics: taxicab and its circles, conics and bisectors.

**Differential geometry** stays in analytic (13.2) and is re-exported here.

**Pictures** in graphics: constructions step by step, the non-Euclidean
models, tilings, polyhedra. The geometry editor (section 2) is the app that
would sit on top of this package.

### 11.3 Dependencies and phases

Depends on core, graphics (the plane kernel, drawing), algebraic and linalg;
the structures package for symmetry groups and proofs; the complex package for
the hyperbolic models; analytic for enclosures.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Euclidean plane: constructions, centres, transformations, checked theorems | 1,200 |
| 2 | Exact computational and lattice geometry, other metrics | 1,100 |
| 3 | Spherical and hyperbolic models | 1,400 |
| 4 | Projective and finite geometries | 1,000 |
| 5 | Solids, symmetry groups, friezes, wallpaper groups, tilings | 1,300 |
| 6 | General proofs of theorems by elimination | 800 |
| 7 | Pictures | 1,000 in graphics |

## 12. A finance package: `@ratmath/finance`

Added on review (James, 2026-11-25): the mathematics of finance, set up so
that it is useful for simulating financial situations, with the commonly
needed pieces only. A full simulation environment would be an app on top.

### 12.1 What exact arithmetic adds

- **Money is exact.** Amounts are rationals; rounding happens only where a
  stated rule says so (to the cent, half up or half even), so a schedule adds
  up to the cent and the final payment is adjusted visibly, not silently.
- **Rates found by solving are certified.** A yield or an internal rate of
  return is the root of an equation. Analytic finds *all* of them, proves each
  one, and says so when a cash flow has more than one, which a spreadsheet's
  single answer hides.
- **Simulated results are labelled**, as in section 6.1, with their seed and
  their sampling error.

### 12.2 Scope

**Time value of money.** Simple, compound and continuous interest; nominal,
effective and annual percentage rates; present and future value; annuities
(ordinary, due, growing, deferred) and perpetuities; solving for any one of
the quantities; day-count conventions and dated cash flows.

**Loans.** Amortisation schedules under a stated rounding rule; extra
payments, balloon and interest-only periods, changes of rate; comparing and
refinancing loans; revolving credit.

**Cash flows.** Net present value, internal rates of return (all of them),
the modified rate, payback periods; inflation, real against nominal amounts;
depreciation methods; taxes as piecewise functions that the caller supplies
(no jurisdiction's tables are built in).

**Bonds and interest rates.** Price, yield to maturity, accrued interest,
duration and convexity; spot and forward rates; a yield curve from bond
prices; immunisation.

**Portfolios.** Returns, risk and covariance from data (statistics package);
the efficient frontier, exactly for rational data (linalg); the capital asset
pricing model.

**Options.** Payoffs and put-call parity; pricing on a binomial tree, exact;
Black–Scholes with certified values and sensitivities.

**Life contingencies.** Life tables, survival probabilities, the present
values of life annuities and insurances.

**Building blocks for simulation.** This is the part James asked for.
- A *scenario*: accounts, dated cash flows (income, spending, contributions,
  payments), and rates that may change over time. Projected step by step,
  exactly, into a table of balances.
- What-if comparison of scenarios (pay off the loan or invest; rent or buy;
  retire at one age or another), and sensitivity: which input moves the
  outcome most, from certified derivatives.
- Uncertain inputs: returns, interest and inflation drawn from the statistics
  package's distributions as independent draws, random walks, geometric
  Brownian motion or mean-reverting rates, or resampled from data the caller
  supplies; one-off random events (an expense, a gap in income) with stated
  probabilities.
- Many runs of a scenario from the seeded generator: the distribution of the
  final balance, the probability of running out, percentile bands over time,
  and the effect of the order in which good and bad years arrive.

No market data, no tax law and no advice are built in; the caller supplies
the numbers.

**Pictures** in graphics: balance and amortisation charts, bands of
percentiles over time, yield curves, option payoffs.

### 12.3 Dependencies and phases

Depends on core, analytic (roots, exponentials), the statistics package
(distributions, the seeded generator) and linalg (portfolios). Phases 1 to 4
need only core and analytic.

| Phase | Contents | Estimate |
|---|---|---|
| 1 | Money and rounding, day counts, time value, annuities | 700 |
| 2 | Loans and schedules, cash flows, rates of return | 800 |
| 3 | Bonds, rates and yield curves | 700 |
| 4 | Scenarios, exact projection, comparison and sensitivity | 700 |
| 5 | Uncertain inputs and repeated runs | 800 |
| 6 | Portfolios, options, life contingencies | 1,100 |
| 7 | Pictures | 500 in graphics |

## 13. Extensions of the current libraries

Added on review (James, 2026-11-25): the rest of differential equations and
the differential geometry of curves and surfaces are to be built into the
current libraries, and the methods of numerical analysis exposed for teaching.
Sequences and series, the theorems of vector calculus and Fourier transforms
were added the same day (13.4 to 13.6). None of these needs a new package.

### 13.1 Differential equations (analytic, linalg, graphics)

Analytic has validated solutions of initial and boundary value problems.
Missing is most of what a course does around them.

- **Solution methods with their steps.** First-order equations of the standard
  types (separable, linear, exact, Bernoulli, homogeneous) and linear equations
  with constant coefficients (characteristic roots as exact algebraic numbers,
  undetermined coefficients, variation of parameters). Each formula found is
  checked against the validated numerical solution, so a wrong closed form
  cannot be returned.
- **Laplace transforms** and their inverses for the course's class of
  functions, using algebraic's partial fractions, including step and impulse
  inputs through analytic's piecewise functions.
- **Linear systems** x′ = Ax solved exactly with linalg's matrix exponential;
  equilibria classified (node, saddle, spiral, centre) from exact eigenvalues.
- **Nonlinear systems in the plane:** equilibria found and certified,
  linearisation, stability, nullclines, and phase portraits in graphics.
- **Series solutions** at ordinary and regular singular points (Frobenius),
  with a proven remainder.
- **Eigenvalue problems** of Sturm–Liouville type, enclosed by shooting with
  the existing boundary value solver.
- **The three classical partial differential equations** (heat, wave, Laplace)
  on an interval, a rectangle and a disc by separation of variables: the
  Fourier coefficients are already certified, and the truncation error of the
  series gets a proven bound. d'Alembert's solution of the wave equation.
  Animated in graphics.

| Part | Where | Estimate |
|---|---|---|
| Solution methods with steps, Laplace transforms | analytic | 1,900 |
| Linear and nonlinear systems, stability | analytic and linalg | 700 |
| Series solutions, Sturm–Liouville | analytic | 900 |
| Heat, wave, Laplace by separation of variables | analytic | 900 |
| Phase portraits, direction of flow, animated solutions | graphics | 800 |

### 13.2 Differential geometry (analytic, graphics)

A subpath of analytic (`@ratmath/analytic/geometry`), built on its fields,
partial derivatives and certified integrals, with the pictures in graphics.

- **Curves** in the plane and in space: arc length, unit tangent, normal and
  binormal, curvature and torsion, the Frenet frame, osculating circles,
  evolutes and involutes, reparametrisation by arc length. Values are
  certified, and closed forms are given where the expressions allow. This also
  supplies the curve features that graphics' "not yet" list asks for.
- **Surfaces:** the first and second fundamental forms, the unit normal, area,
  Gaussian, mean and principal curvatures, the classification of points
  (elliptic, hyperbolic, parabolic, umbilic), lines of curvature and
  asymptotic directions.
- **Intrinsic geometry:** Christoffel symbols, geodesics as validated ODEs,
  parallel transport and holonomy, geodesic curvature.
- **Gauss–Bonnet as a checked statement:** the certified integral of the
  curvature over a closed surface is an interval that must contain 2π times
  the Euler characteristic, and does.
- **Pictures:** a frame moving along a curve (animation), surfaces coloured by
  curvature, geodesics and transported vectors drawn on a surface.

| Part | Where | Estimate |
|---|---|---|
| Curves | analytic | 500 |
| Surfaces, fundamental forms, curvatures | analytic | 600 |
| Geodesics, transport, Gauss–Bonnet | analytic | 500 |
| Pictures | graphics | 700 |

### 13.3 Numerical analysis for teaching (analytic, linalg, core, graphics)

The libraries avoid the classical methods on purpose: they return proven
answers. That is what makes them a good place to study those methods. Each
method becomes an object that shows its steps, and because the certified
answer is available, its *true* error is shown beside the textbook error
estimate, at every step.

- **Floating point as a subject.** Every double is an exact rational, which
  Core can hold. So rounding error can be computed exactly: the exact value of
  each float, the error of each operation, cancellation and its repair, error
  growing through a calculation.
- **Root finding:** bisection, fixed-point iteration, Newton, secant, with
  iteration tables, observed order of convergence, and pictures (cobweb
  diagrams, Newton's tangents; basins of attraction once the complex package
  exists).
- **Interpolation and approximation:** Lagrange and Newton forms with exact
  divided differences, Hermite interpolation, splines, Chebyshev nodes, the
  Runge phenomenon, least-squares polynomials; the error formula against the
  true error.
- **Differentiation and quadrature:** difference formulas, Richardson
  extrapolation, trapezoid, Simpson, Romberg, Gauss rules with their nodes as
  exact algebraic numbers, adaptive rules.
- **Methods for differential equations:** Euler, Heun, Runge–Kutta, multistep
  methods, with step tables, global error against the validated solution, step
  size experiments and stability regions.
- **Numerical linear algebra** in linalg: Jacobi, Gauss–Seidel, successive
  over-relaxation, conjugate gradients, the power method and QR iteration,
  the effect of pivoting, condition numbers; each run in floats and in exact
  rationals side by side, so the part of the error due to rounding is
  separated from the part due to the method.

| Part | Where | Estimate |
|---|---|---|
| Exact model of floating-point arithmetic | core or analytic | 400 |
| Root finding, interpolation, quadrature, differentiation | analytic (`@ratmath/analytic/methods`) | 1,400 |
| Methods for differential equations | analytic | 600 |
| Iterative and eigenvalue methods | linalg | 800 |
| Pictures | graphics | 500 |

### 13.4 Sequences and series (analytic)

- Sequences with certified limits; monotone and bounded sequences; recursive
  sequences (shared with section 10).
- Convergence tests as named steps (comparison, ratio, root, integral,
  alternating), each returning convergent, divergent or inconclusive with the
  quantities that decided it, and an enclosure of the sum with a proven tail
  bound.
- Power series: radius and interval of convergence with the end points
  examined, operations on series, Taylor's remainder.
- Sequences of functions: pointwise against uniform convergence shown with
  enclosures of the supremum of the difference.

About 800 lines. Serves calculus II and the computational side of real
analysis.

### 13.5 The theorems of vector calculus (analytic)

Green's, Stokes' and the divergence theorem as checked identities (both sides
enclosed, and the enclosures must meet); change of variables with the Jacobian
as a visible step; conservative fields and potentials; Lagrange multipliers,
with every candidate point certified. About 600 lines.

### 13.6 Fourier transforms (analytic)

These belong beside the Fourier series analytic already has (James,
2026-11-25).

- The Fourier transform and its inverse with certified values, on the existing
  improper integrals; the standard pairs and rules; convolution; Parseval's
  identity as a checked statement.
- The discrete Fourier transform, exactly over the roots of unity from
  algebraic and as enclosures; the fast algorithm shown beside the definition;
  sampling and aliasing.
- Use with section 13.1: the heat and wave equations on the whole line.

About 900 lines.

## 14. Coverage of an undergraduate curriculum

"Covered" here means: the calculations a course asks a student to carry out
can be done in the JavaScript libraries with an exact or certified answer, and
most have a picture or a worked form. It does not mean proofs, which are the
point of the upper-level courses and which no computing library supplies. The
percentages are a judgement from each course's usual list of topics, not a
measurement. The RiX column records that a plugin exists; those were assessed
from their READMEs only.

| Course | Libraries today | RiX plugins today | After everything planned here | Still missing |
|---|---|---|---|---|
| College algebra and precalculus | 50% | yes | 90% | Word problems; the full range of textbook step conventions |
| Calculus I and II | 90% | yes | 97% | Antiderivatives outside the taught methods |
| Calculus III (several variables, vector calculus) | 80% | part | 93% | Differential forms |
| Linear algebra | 95% | yes | 95% | Symbolic parameters (section 2); abstract vector spaces and inner products beyond R^n |
| Differential equations (ordinary) | 70% | part | 90% | Delay and stochastic equations, bifurcation theory, chaos beyond pictures |
| Complex analysis | 10% | 15% | 85% | Riemann mapping in general, analytic continuation beyond named paths, harmonic functions and Dirichlet problems |
| Number theory | 40% | little | 90% | Ranks of elliptic curves, the deeper analytic theory |
| Abstract algebra I (groups) | 30% | none | 85% | Infinite groups beyond presentations, representation theory beyond character tables |
| Abstract algebra II (rings, fields, Galois) | 60% | part | 90% | Galois groups of high degree, commutative algebra beyond Gröbner bases, modules over general rings |
| Real analysis | 30% | part | 45% | The proofs, which are the course; see section 15 |
| Numerical analysis | 50% | part | 85% | Numerical methods for partial differential equations, large sparse problems, fast transforms |
| Probability | 0% | yes | 85% | Measure-theoretic foundations; stochastic processes beyond Markov chains |
| Statistics | 0% | yes | 80% | Generalised linear models beyond logistic, time series, multivariate methods, experimental design |
| Discrete mathematics and combinatorics | 10% | part | 85% | Asymptotic and analytic combinatorics; proof techniques as such |
| Graph theory | 0% | part | 80% | Hard problems beyond small graphs; extremal and random graph theory as theory |
| Game theory | 0% | none found | 75% | Games of incomplete information, repeated and evolutionary games, mechanism design |
| Geometry | 50% | yes | 90% | The editor, which is an app |
| Dynamical systems and chaos | 5% | part | 85% | Ergodic theory; higher-dimensional and infinite-dimensional systems |
| Fourier analysis | 30% | none | 75% | Distributions, wavelets |
| Logic and set theory | 0% | part | 80% | Completeness and incompleteness as theorems; axiomatic set theory beyond ordinal and cardinal arithmetic |
| Theory of computation | 0% | none found | 70% | Complexity theory; undecidability as theorems |
| Topology | 0% | none | 5% | See section 15 |
| Differential geometry | 10% | none | 80% | Manifolds, tensors and differential forms in general; curvature in higher dimensions |
| Partial differential equations | 10% | none | 55% | Characteristics, Green's functions, transforms on unbounded domains, nonlinear equations, validated numerical solvers |
| Financial mathematics | 0% | none found | 85% | Stochastic calculus proper; market conventions of particular countries |
| Optimisation | 20% | yes | 65% | General nonlinear and convex optimisation |

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
The logic package covers the one remaining course in which students compute
by fixed rules.

## 15. What would remain

With sections 3 to 13 built, every course in the table of section 14 that has
calculations in it is planned for. What is left:

1. **Nonlinear and convex optimisation** beyond Lagrange multipliers. RiX has
   an `optimize` plugin.
2. **Smaller subjects** with something to compute and no plan: the calculus
   of variations, tensors and multilinear algebra, stochastic processes beyond
   Markov chains, numerical methods for partial differential equations.
3. **Courses that are about proof, not calculation.** Real analysis, topology,
   measure theory and functional analysis. What computing can offer real
   analysis is the project's own subject (reals as families of rational
   intervals) together with section 13.4. Point-set topology offers little to
   compute; finite simplicial homology over Z is computable through the Smith
   normal form and would be a small addition (about 600 lines) rather than
   coverage of the course.
4. **The geometry editor**, which is an app.

## 16. Order, and decisions made

Suggested order, with the reason for each step. Several steps name more than
one package because they do not depend on each other.

1. **Done 2026-11-25.** Several-variable polynomials (section 5, phase 2),
   because linalg's symbolic parameters, graphics' higher curves, the
   symbolic package and geometric proofs all wait on them.
2. **Done 2026-11-26.** Symbolic phase 1 (expressions, rewriting, equality), because the solving
   and step-by-step features of several later sections use it.
3. **Done 2026-11-26.** Complex phases 1 and 2 (numbers, branches, quaternions and octonions).
   Self-contained, and they settle the exact complex type that linalg's
   complex entries, the hyperbolic models and the Julia sets need.
4. **Done 2026-11-26.** Discrete phase 1 (counting, enumeration, the seeded generator), because
   statistics builds on it; logic phases 1 and 2, which depend on nothing.
5. Statistics phases 1 to 3. The most widely used subject, and enough for the
   first experiments with generated data.
6. Symbolic phases 2 and 3 (solving, antiderivatives), with sections 13.4 and
   13.5. Together these finish the calculus sequence and college algebra.
7. Chaos phases 1 and 2, geo phases 1 and 2 and finance phases 1 to 4. Each
   needs only what exists today, and each shows off exact arithmetic quickly.
8. Discrete phase 2 (graphs) and number theory phases 1 to 3.
9. Abstract algebra phases 1 and 3 (groups and rings).
10. Statistics phases 4 and 5 (the standard tests and regression) and discrete
    phase 5 (linear programming and matrix games).
11. Complex phases 3 and 4 (the analysis proper). The hardest single piece.
12. The remaining phases as interest directs.

The other extensions of section 13 touch only the current libraries and can be
taken up at any point between these steps. Differential geometry of curves and
the exact model of floating point are the smallest self-contained starts.

Taken together, sections 3 to 13 come to roughly 85,000 lines by these
estimates, about three times the size of the Shelf today.

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

The rest of differential equations and differential geometry are built into
the current libraries, and numerical analysis is exposed there for teaching
(section 13; James, 2026-11-25).

Formal logic becomes a library, `@ratmath/logic` (section 8; James,
2026-11-25).

Symbolic algebra, dynamical systems and geometry become libraries,
`@ratmath/symbolic`, `@ratmath/chaos` and `@ratmath/geo` (sections 9 to 11),
and sequences and series, the theorems of vector calculus and Fourier
transforms go into analytic (13.4 to 13.6; James, 2026-11-25).

The mathematics of finance becomes a library, `@ratmath/finance`, with the
commonly needed building blocks for simulating financial situations; a full
simulation environment would be an app (section 12; James, 2026-11-25).

## 17. How every package is built, and where progress is recorded

Decided by James, 2026-11-25.

**Documentation.** Every new package is built the way the current Shelf
libraries are, with a `docs/` folder published at `ratmath.com/<package>`
that contains:

- an **overview** page (`index.html`): what the package is for, with a first
  example;
- an **exploration** page (`explore.html`): an interactive page for trying
  the package without writing code;
- a **detailed** page (`reference.html`): every function, its arguments and
  what it returns;
- a **separate page for each topic**, giving the usage and the mathematics
  behind it, and ending with a runnable code box seeded with some interesting
  code on that topic.

The pages use the shared pieces the libraries already have (`runbox.js`,
`docs-kit.js`, KaTeX from `ratmath.com/assets`), and the docs build runs every
page's code box and fails on any error. Each package also ships its tests,
TypeScript declarations, a README and a changelog, as the current ones do.

**Where progress is recorded.** This document is the plan. The public record
of what exists is the RatMath Shelf page on ratmath.com, which lists every
Shelf package and what each can do, with links to its documentation. When a
topic here is implemented, it moves from this document to that page: the
entry is removed or marked done here and added there with a link to the
package's documentation page for it. The homepage keeps Core as its
foundation and links to the Shelf page.

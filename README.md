# RatMath

This is an umbrella repo tracking various parts of the RatMath ecosystem. RatMath is an attempt to explore rational mathematics and implement real numbers as rational interval families, specifically in an oracle fashion. 

The various parts are: 

## Documentation

- [Consolidated execution plan](WORK_PLAN.md): ordered implementation tasks, completion checks, and deferred decisions.

- [Language Guide](docs/LANGUAGE_GUIDE.md): Detailed guide on variables, functions, scoping, and strictness rules.
- [JS Integration](docs/JS_INTEGRATION.md): How to integrate JS modules.
- [Library roadmap](docs/design/library-roadmap.md): what the JavaScript libraries cover, what is left to build, proposed complex, number theory, abstract algebra, statistics, discrete mathematics, logic, symbolic algebra, dynamical systems, geometry and finance packages, extensions of the current libraries, and coverage of the undergraduate curriculum.

## Workspace setup

The repository standardizes on Bun 1.4 through both `.bun-version` and the
root `packageManager` field. Git submodules and Bun workspaces solve different
problems: the submodule revisions define the compatible integration set, while
Bun links packages at those checked-out revisions. A workspace install does not
move a submodule to a newer commit.

Use the read-only check in automation or before changing anything:

```sh
bun run prep:check
```

Use `bun run prep` in a terminal for the guided version. It reports each
umbrella revision, checkout, branch, cached ahead/behind state, and dirty files.
It asks before restoring any clean submodule to its compatible pin and refuses
to change dirty repositories. For clean branch checkouts, it can then fetch
remote refs and offer only a fast-forward update, with a separate confirmation
for each repository; such an update deliberately appears as a changed
submodule revision for review in the umbrella. It separately asks before
running locked dependency installs.

## Test profiles

Run these commands from the umbrella root:

```sh
bun run test:short          # broad edit loop, 29.7 seconds on the reference run
bun run test:ten            # medium confidence, targeted at five to ten minutes
bun run test:suite          # every Bun test in every relevant repository
bun run test:plugin plot    # one RiX plugin and the tests that load it
```

`bun run test` is an alias for `test:suite`. Direct `bun test` remains the
comprehensive automatic-discovery check; the root `bunfig.toml` only excludes
the ignored `tmp/` diagnostics directory and the copies that Shelf libraries
vendor for standalone use (`shelf/*/vendor/`). The named profiles are preferable in
scripts because they execute repositories sequentially and identify the one
that failed.

## RatMath Shelf

The RatMath Shelf is the family of subject libraries built on Core. Each one is
its own public repository and npm package under `@ratmath/`, pinned here as a
submodule in `shelf/`. Each has a docs site at `ratmath.com/<name>/`, and the
[Shelf page](https://ratmath.com/shelf/) lists what each one can do so far.

| Library | npm | What it does |
|---------|-----|--------------|
| [`shelf/algebraic`](https://github.com/ratmath/algebraic) | `@ratmath/algebraic` | Exact algebraic numbers, polynomial factoring, Galois groups, number and finite fields |
| [`shelf/analytic`](https://github.com/ratmath/analytic) | `@ratmath/analytic` | Certified calculus: values, ranges, integrals, zeros, limits, Taylor models, ODE steps |
| [`shelf/chaos`](https://github.com/ratmath/chaos) | `@ratmath/chaos` | Dynamical systems: orbits, periodic points, bifurcations, Lyapunov exponents, fractals |
| [`shelf/complex`](https://github.com/ratmath/complex) | `@ratmath/complex` | Exact complex arithmetic, honest branches, certified complex analysis, quaternions, octonions |
| [`shelf/discrete`](https://github.com/ratmath/discrete) | `@ratmath/discrete` | Counting and enumeration, graphs, generating functions, games, exact linear programming |
| [`shelf/finance`](https://github.com/ratmath/finance) | `@ratmath/finance` | Exact money, time value, loans, bonds, scenarios, building blocks for financial simulation |
| [`shelf/geo`](https://github.com/ratmath/geo) | `@ratmath/geo` | Euclidean constructions with checked theorems; spherical, hyperbolic, projective and finite geometries |
| [`shelf/graphics`](https://github.com/ratmath/graphics) | `@ratmath/graphics` | Certified plots, regions and exact geometry, exported to SVG, TikZ, PDF and PNG |
| [`shelf/linalg`](https://github.com/ratmath/linalg) | `@ratmath/linalg` | Exact rational linear algebra with float previews that sharpen to exact |
| [`shelf/logic`](https://github.com/ratmath/logic) | `@ratmath/logic` | Propositional and predicate logic, proof checking, model search, modal logic, computability |
| [`shelf/numtheory`](https://github.com/ratmath/numtheory) | `@ratmath/numtheory` | Primes with certificates, factoring, congruences, Diophantine equations, p-adics, elliptic curves, class groups, zeta and L values |
| [`shelf/stats`](https://github.com/ratmath/stats) | `@ratmath/stats` | Exact finite probability, certified distributions, seeded data with noise, tests, regression |
| [`shelf/structures`](https://github.com/ratmath/structures) | `@ratmath/structures` | Abstract algebra: finite groups and character tables, polynomials in several variables, Gröbner bases, Galois groups, chain complexes |
| [`shelf/symbolic`](https://github.com/ratmath/symbolic) | `@ratmath/symbolic` | Expressions, rewriting, solving and closed forms, each step checked by the certified libraries |

The layers are: Core (`core/`) holds the number types, the Shelf holds
the subjects, and RiX is the language that calls them. Apps sit on top.

Many Shelf libraries vendor their siblings as submodules (for example graphics
vendors analytic and algebraic, stats vendors analytic and discrete, symbolic
vendors analytic, algebraic and structures) so they build on their own.
In this umbrella those copies are not used: Bun only reads the root
`workspaces`, so every library links to the sibling in `shelf/`. The umbrella
pins are the set that has to work together; bump them here.


# Satisfiability Overview of Predicates for ProB Backends

This directory contains gathered data for predicates over the B method.
These predicates were considered as constraint problems and
given to the ProB backends listed below to find a solution.
Stored for each predicate is the time needed per backend
to determine whether a solution exists or not,
or `-1` if the backend was not able to provide an answer.

At first, I present the results, as they represent the main reason to put this
repository together. Secondly, I describe the generation step of the considered
formulae. And finally, I describe the file format of the individual results.

Below, the performances of the backends will be briefly compared.
Further, it will be outlined how the predicates were acquired in the first
place, and how the file format is set up.

## Accounted Solvers

The solvers used for this were the following ProB backends:

- ProB CLP(FD) [native ProB backend, further referenced to as ProB]
- KodKod
- Z3
- the SMT\_SUPPORTED\_INTERPRETER setting of ProB, which is essentially a
  combination of ProB and Z3

## Measured Results

Comparison of ProB, KodKod, Z3, and the SMT\_SUPPORTED\_INTERPRETER setting of
how many predicates they were able to determine satisfiablility for,
and how they compare to each other.

Additionally, the time needed for those predicates the backends
could return an answer for was measured for each solver respectively.
Time measurements are given in nano seconds (`10^-9` seconds) as mean of three
runs.

In total, this takes `321742` predicates into account. See the
**Accounted Formulae** section below for more details on this.

### Overview for ProB's CLP(FD) Backend

297774 of 321742 samples could be decided (0.9255055292750092 of all samples):

- 297180 could also be decided by KODKOD (0.9980051985734147 of decided samples)
- 148218 could also be decided by Z3 (0.49775332970642167 of decided samples)
- 294485 could also be decided by SMT\_SUPPORTED\_INTERPRETER (0.988954710619463 of decided samples)

Statistics:

- Minimum: 1.2768733E7, Maximum: 2.1827261325733334E11
- Mean: 1.4374553970685193E8
- Variance: 6.7062798451197709E17
- Standard deviation: 8.18918789937059E8
- Median: 5.8337225166666664E7
- First Quartile: 5.0893855333333336E7, Third Quartile: 8.6140214E7

### Overview for KODKOD

298383 of 321742 samples could be decided (0.9273983502309304 of all samples):

- 297180 could also be decided by PROB (0.9959682689697469 of decided samples)
- 148422 could also be decided by Z3 (0.4974210997275314 of decided samples)
- 294105 could also be decided by SMT\_SUPPORTED\_INTERPRETER (0.9856627220719679 of decided samples)

Statistics:

- Minimum: 2.890165E7, Maximum: 2.2012266534833334E11
- Mean: 2.4473987603919712E8
- Variance: 8.7777435755771776E17
- Standard deviation: 9.368961295457025E8
- Median: 7.268096033333333E7
- First Quartile: 5.9440985333333336E7, Third Quartile: 1.4169543966666666E8

### Overview for Z3

156412 of 321742 samples could be decided (0.48614106955262293 of all samples):

- 148218 could also be decided by PROB (0.947612715136946 of decided samples)
- 148422 could also be decided by KODKOD (0.9489169628928726 of decided samples)
- 148245 could also be decided by SMT\_SUPPORTED\_INTERPRETER (0.9477853361634657 of decided samples)

Statistics:

- Minimum: 1.3836093E7, Maximum: 1.3207398927733333E11
- Mean: 1.6464577995095307E8
- Variance: 5.7431301212307142E17
- Standard deviation: 7.578344226300831E8
- Median: 7.334618683333334E7
- First Quartile: 6.452551233333333E7, Third Quartile: 1.0273993866666666E8

### Overview for SMT\_SUPPORTED\_INTERPRETER

294662 of 321742 samples could be decided (0.915833183109448 of all samples):

- 294485 could also be decided by PROB (0.999399311753806 of decided samples)
- 294105 could also be decided by KODKOD (0.9981096985698868 of decided samples)
- 148245 could also be decided by Z3 (0.5031018590792162 of decided samples)

Statistics:

- Minimum: 3.0185891333333332E7, Maximum: 1.3123958382166667E11
- Mean: 3.29716292411948E8
- Variance: 8.4224374876478093E17
- Standard deviation: 9.177383879759966E8
- Median: 1.1596152216666666E8
- First Quartile: 6.827432433333333E7, Third Quartile: 2.6445271666666666E8

## Accounted Formulae

In total, `321742` formulae were taken into account,
which were constructed from the `prob_examples/public_examples`
(commit `1b18afd153633dc672803c1b8fc9f2d05c5c40a1`)
by the following steps:

- get data from source machine file (.mch and .bcm)
    - properties `P` (conjunction of all properties)
    - invariants `I` (conjunction of all invariants `i`)
    - guards `G(E)` for each event `E`
    - assertions and theorems `A`
    - weakest preconditions `WK(i,G)` for each invariant `i` and guard `G`
    - before/after predicates `BA(E)` for each event `E`
- add (constructed) formulae
    - add invariants and properties
    - construct from guards for each event `E`
        1. `P & I & G(E)`
        2. `not(P & I) => G(E)`
        3. `P & I & not(G(E))`
        4. `not(P & I) => not(G(E))`
        5. `not(P & G(E)) => I`
        6. `not(P & not(G(E)) => I`
        7. `P & not(I) & G(E)`
        8. `not(P & not(I)) => G(E)`
        9. `P & not(I) & not(G(E))`
        10. `not(P & not(I)) => not(G(E))`
        11. `not(P & not(G(E)) => not(I)`
        12. `not(P & G(E)) => not(I)`
    - construct from assertions/theorems `A`
        1. `A`,
        2. `P & I & A`
        3. `P & I & not(A)`
        4. `not(P & I) => A`
        5. `P & I & As` with `As` the conjunction of all `A`
        6. `P & I & not(As)` with `As` the conjunction of all `A`
        7. `not(P & I) => As` with `As` the conjunction of all `A`
    - construct from guards `G1:=G(E1)`, `G2:=G(E2)` for each pair of unequal
    events `(E1, E2)`
        1. `P & I & G1 & G2`
        2. `P & I & (not(G1) => G2)`
        3. `P & I & G1 & not(G2)`
        4. `P & I & (G1 => not(G2))`
    - construct from guards `G1:=G(E1)`, `G2:=G'(E2)` with `G'(E)` being the
    primed version of the guard `G(E)`, and before/after predicates
    `BA:=BA(E1)` for each pair of unequal events `(E1, E2)`
        1. `P & I & G1 & BA & G2`
        2. `P & I & G1 & BA & not(G2)`
        3. `P & I & not(G1 & BA) & G2`
        4. `P & I & not(G1 & BA) & not(G2)`
        5. `P & I & (not(G1 & BA) => G2)`
        6. `P & I & (not(G1 & BA) => not(G2))`
    - construct from the weakest preconditions `WK:=(i,G(E))` for each
    invariant `i` and event `E`
        1. `P & I & WK`
        2. `P & I & not(WK)`
        3. `P & (not(I) => WK)`
        4. `P & (not(I) => not(WK))`
    - construct from the before/after predicates `BA:=BA(E)`, guards
    `G:=G(E)` for each event `E`,
    and a single invariant `i`, and`j:=i'` (primed invariant)
        1. `P & i & G & BA & j`
        2. `P & i & G & BA & not(j)`
        3. `P & (not(i & G & BA) => j)`
        4. `P & (not(i & G & BA) => not(j))`

### Completeness of the Gathered Formulae

The mapping from machine file to all of the gathered predicates described above
does not work for all files. For some machines the generation of the weakest
preconditions or the before/after predicates resulted partially in exceptions,
so they were simply disregarded.

Some machines could not be correctly loaded in the first place
(used ProB2 for this),
so those were completely disregarded as well.

If a solver needed more than 2.5 seconds for an individual
predicate, the backend timeouted, resulting in no answer.

Eventually, 2272 of the public ProB examples were taken into account.

## Accessing the Individual Results

If you feel merry and want to work with those results,
e.g. for a deeper analysis of the individual solvers, all individual
measurements are gathered in their own files, found in the `public_examples/`
directory.

The directory hierarchy resembles the one of the `prob_examples/` repository,
so that each machine file can easily be mapped to the corresponding
results-file.
Each results-file contains all generated formulae from the corresponding
machine file.

### File Format

The format of the individual files is pretty straight forward.

    <Result File> ::= <Source annotation> <Result List>
    <Source annotation> ::= #source: <path to source machine file> '\n'
    <Result List> ::= <Result> | <Result> '\n' <Result List>
    <Result>      ::= <Time> ',' <Time> ',' <Time> ',' <Time> ':' <predicate>
    <Time>        ::= -1 | <needed time>

The annotated source path corresponds to the path of the source machine file
in the `prob_examples` repository. To be correct, it states a path, that assumes
the presence of the examples in `examples/prob_examples/`.

The stated time records correspond, in this order, to ProB, KodKod, Z3,
SMT\_SUPPORTED\_INTERPRETER.
They are the mean of three runs per predicate, for each solver, and stated in
nano seconds in double precision. `-1` indicates that the respective solver
could *not* decide the predicate.

#### Example File

    #source:examples/prob_examples/public_examples/path/to/source.mch
    -1,-1,1.23456789E8,2.3456789E8:(x<y) & (y<x)
    -1,-1,1.23456789E8,2.3456789E8:(x<y) <=> (y<x)

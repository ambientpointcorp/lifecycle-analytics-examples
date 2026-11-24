# lifecycle-analytics-examples

Worked examples of [`dwellr`](https://github.com/ambientpointcorp/dwellr), one
script per chapter of *Lifecycle Monitoring Analytics: A toolkit for continuous
supply chain operations*, which develops the package's methods.

Install the package first. Every script here calls it rather than
reimplementing any of it.

```r
# install.packages("remotes")
remotes::install_github("ambientpointcorp/dwellr")
```

## Layout

```
scripts/   one file per chapter and per appendix, generated from the manuscript
recorded/  worked examples on data the package did not generate, hand-written
R/         the plot theme and figure helper every script shares
out/       figures, regenerated rather than tracked
data/      the one cached artifact the scripts share, if you choose to cache it
```

## Running a chapter

Run from the repository root, so that `source("R/theme.R")` resolves.

```bash
Rscript scripts/14-setting-thresholds.R
```

Each script is self-contained: it loads the packages that chapter needs, draws
the corpus, and prints the figures and tables the chapter prints. No script
sources another, so a reader interested in one chapter runs one file.

`make list` names the scripts. `make run` runs all of them and reports which
fail, which takes roughly half an hour: Chapter 9's model fits and Chapter 20's
echelon searches dominate the total.

## Measuring a lifecycle the book did not generate

Every script in `scripts/` starts from `simulate_lifecycles()`, so every one of
them measures a corpus whose stages are the book's own: `received`, `released`,
`shipped`. An operation measured for real has its own stage names and its own
set of endings, and the package reads both from a `stage_spec()` rather than
assuming them.

`recorded/` shows that path end to end on a freight lifecycle that shares no
stage name with the book's: declaring the stage set, building a population from
a three-column event log with `as_lifecycle()`, validating the log against the
specification, and then the same dwell estimation, thresholds and lead-time
decomposition the chapters perform.

```bash
Rscript recorded/measuring-a-recorded-log.R
```

It closes by measuring the same population twice, once against its own
specification and once against the book's, because the failure it guards
against raises no error: a stage set that does not declare `departed` as an
ending reads every departure as a wait that never finished, and the stage
where the work completes becomes the worst-performing stage in the operation.

These files are hand-written, the one exception to the rule in the next
section. They illustrate the package rather than a chapter, so there is no
manuscript chunk to generate them from.

## The scripts are generated

The extractor lives in the manuscript, beside the chapter sources it reads,
and writes into a checkout of this repository. Run it from there:

```bash
cd ../lifecycle-analytics-manuscript
python3 assets/extract.py ../lifecycle-analytics-examples
```

A hand-written example can drift from the chapter it illustrates, and the
drift surfaces only when a number disagrees with the page. A generated one
cannot drift, because the chapter is its only source. Do not edit anything
under `scripts/` directly: the next extraction overwrites it.

The manuscript repository is private until publication, so the extractor can
be run only by someone with a checkout of it; `make extract` prints the
command rather than running it. The generated scripts are committed here for
that reason.

## The corpus

Every script draws its data from `simulate_lifecycles()` in `dwellr`, at
`n = 5000, seed = 11` for most chapters. The generator is deterministic given
its seed, so a figure here reproduces the figure in print exactly.

The corpus deliberately carries the pathologies the book has to measure:
right-censoring from an observation time inside the horizon, two-component
dwell distributions, partial fills, loops, open-ticket residue, and covariate
effects on dwell. Appendix B of the book lists what it does not reproduce:
clean identifiers, no seasonality, no non-stationarity, and two branches that
behave alike. Each of those is a reason to re-measure on your own data rather
than carry a figure across.

## Your own extract

The estimators depend on the shape of the tables rather than on the generator.
Two data frames are enough for all of Part II: an `orders` frame with an
identifier, an entry timestamp, the current stage, a terminal flag and any
cohort keys; and an `events` frame with `order_no`, `event`, `stage` and
`occurred_at`. Supply `as_of` as the extract time and `stage_dwell()` will set
the censoring flag correctly.

Appendix B of the book works a two-order example end to end.

## License

MIT. See [LICENSE](LICENSE). The book itself is copyrighted separately.

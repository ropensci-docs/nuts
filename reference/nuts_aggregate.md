# Aggregate to higher order NUTS levels

`nuts_aggregate()` transforms regional NUTS data between NUTS levels.

## Usage

``` r
nuts_aggregate(
  data,
  to_level,
  variables,
  weight = NULL,
  missing_rm = FALSE,
  missing_weights_pct = FALSE,
  multiple_versions = c("error", "most_frequent")
)
```

## Arguments

- data:

  A nuts.classified object returned by
  [`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md).

- to_level:

  Number corresponding to the desired NUTS level to be aggregated to:
  `1` or `2`.

- variables:

  Named character specifying variable names and variable type
  (`'absolute'` or `'relative'`), e.g. `c('var_name' = 'absolute')`.

- weight:

  String with name of the weight used for conversion. Can be area size
  `'areaKm'` (default), population in 2011 `'pop11'` or 2021 `'pop21'`,
  artificial surfaces in 2012 `'artif_surf12'` and 2018 `'artif_surf18'`
  or residential built-up volume 2021 `'bu_vol'`.

- missing_rm:

  Boolean that is FALSE by default. TRUE removes regional flows that
  depart from missing NUTS codes.

- missing_weights_pct:

  Boolean that is FALSE by default. TRUE computes the percentage of
  missing weights due to missing departing NUTS regions for each
  variable.

- multiple_versions:

  By default equal to `'error'`, when providing multiple NUTS versions
  within groups. If set to `'most_frequent'` data is converted using the
  best-matching NUTS version.

## Value

A tibble containing NUTS codes, aggregated variable values, and possibly
grouping variables.

## Details

Console messages can be controlled with
`rlang::local_options(nuts.verbose = "quiet")` to silence messages and
`nuts.verbose = "verbose"` to switch messages back on.

## Examples

``` r
library(dplyr)
#> 
#> Attaching package: ‘dplyr’
#> The following objects are masked from ‘package:stats’:
#> 
#>     filter, lag
#> The following objects are masked from ‘package:base’:
#> 
#>     intersect, setdiff, setequal, union

# Load EUROSTAT data of manure storage deposits
data(manure)

# Data varies at the NUTS level x indicator x year x country x NUTS code level
head(manure)
#> # A tibble: 6 × 4
#>   indic_ag   geo    time values
#>   <chr>      <chr> <dbl>  <dbl>
#> 1 I07A1_EQ_Y AT     2010  97401
#> 2 I07A1_EQ_Y AT1    2010  21388
#> 3 I07A1_EQ_Y AT11   2010   2110
#> 4 I07A1_EQ_Y AT111  2010    270
#> 5 I07A1_EQ_Y AT112  2010    455
#> 6 I07A1_EQ_Y AT113  2010   1385

# Aggregate from NUTS 3 to 2 by indicator x year
manure %>%
  filter(nchar(geo) == 5) %>%
  nuts_classify(nuts_code = "geo",
                group_vars = c('indic_ag','time')) %>%
  # Group vars are automatically passed on
  nuts_aggregate(to_level = 2,
                 variables = c('values'= 'absolute'),
                 weight = 'pop21')
#> 
#> ── Classifying version of NUTS codes ───────────────────────────────────────────
#> Within groups defined by country, indic_ag, and time:
#> ! These NUTS codes cannot be identified or classified: ME000, ME000, ME000,
#>   ME000, ME000, ME000, ME000, ME000, NOZZZ, NOZZZ, NOZZZ, NOZZZ, SIZZZ, and
#>   NOZZZ.
#> ✔ Unique NUTS version classified.
#> ✖ Missing NUTS codes detected. See the tibble 'missing_data' in the output.
#> 
#> ── Aggregating level of NUTS codes ─────────────────────────────────────────────
#> Within groups defined by country, indic_ag, and time:
#> ℹ Aggregate from NUTS regional level 3 to 2.
#> ✖ These NUTS codes cannot be converted and are dropped: ME000, NOZZZ, and
#>   SIZZZ.
#> ✔ Version is unique.
#> ✖ Missing NUTS codes in data. No values are calculated for regions associated
#>   with missing NUTS codes. Ensure that the input data is complete.
#> # A tibble: 3,032 × 5
#>    to_code country indic_ag     time values
#>    <chr>   <chr>   <chr>       <dbl>  <dbl>
#>  1 AT11    Austria I07A1_EQ_Y   2010   2110
#>  2 AT11    Austria I07A1_EQ_YC  2010    337
#>  3 AT11    Austria I07A2_EQ_Y   2010    802
#>  4 AT11    Austria I07A2_EQ_YC  2010    756
#>  5 AT11    Austria I07A31_EQ_Y  2010    278
#>  6 AT11    Austria I07A32_EQ_Y  2010     31
#>  7 AT11    Austria I07A3_EQ_YC  2010    261
#>  8 AT11    Austria I07A_EQ_Y    2010   2180
#>  9 AT12    Austria I07A1_EQ_Y   2010  19250
#> 10 AT12    Austria I07A1_EQ_YC  2010   1853
#> # ℹ 3,022 more rows
```

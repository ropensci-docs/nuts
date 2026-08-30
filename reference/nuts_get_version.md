# Return version overlap of classified NUTS data

`nuts_get_version()` returns the classified data after running
[`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md).

## Usage

``` r
nuts_get_version(data)
```

## Arguments

- data:

  A nuts.classified object returned by
  [`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md).

## Value

A tibble that lists the group-specific overlap with each NUTS version.

## Details

Console messages can be controlled with
`rlang::local_options(nuts.verbose = "quiet")` to silence messages and
`nuts.verbose = "verbose"` to switch messages back on.

## Examples

``` r
library(dplyr)

# Load EUROSTAT data of manure storage deposits
data(manure)

# Classify version of NUTS 2 codes in Germany
classified <- manure %>%
   filter(nchar(geo) == 4) %>%
   filter(indic_ag == 'I07A_EQ_Y') %>%
   filter(grepl('^DE', geo)) %>%
   filter(time == 2003) %>%
   select(-indic_ag, -time) %>%
   # Data varies at the NUTS code level
   nuts_classify(nuts_code = 'geo')
#> 
#> ── Classifying version of NUTS codes ───────────────────────────────────────────
#> Within groups defined by country:
#> ! These NUTS codes cannot be identified or classified: DEZZ.
#> ✔ Unique NUTS version classified.
#> ✖ Missing NUTS codes detected. See the tibble 'missing_data' in the output.

nuts_get_version(classified)
#> # A tibble: 7 × 3
#>   from_version country overlap_perc
#>   <chr>        <chr>          <dbl>
#> 1 2006         Germany        100  
#> 2 2024         Germany         88.6
#> 3 2021         Germany         88.6
#> 4 2016         Germany         88.6
#> 5 2013         Germany         88.6
#> 6 2010         Germany         88.6
#> 7 NA           NA              NA  
```

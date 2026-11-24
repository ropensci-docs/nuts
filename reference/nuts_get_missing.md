# Return missing NUTS codes in classified NUTS data

`nuts_get_missing()` returns the classified data after running
[`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md).

## Usage

``` r
nuts_get_missing(data)
```

## Arguments

- data:

  A nuts.classified object returned by
  [`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md).

## Value

A tibble listing missing NUTS codes for each group.

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

nuts_get_missing(classified)
#> # A tibble: 4 × 4
#>   from_code from_version from_level country
#>   <chr>     <chr>             <dbl> <chr>  
#> 1 DE30      2006                  2 Germany
#> 2 DE50      2006                  2 Germany
#> 3 DE60      2006                  2 Germany
#> 4 DEC0      2006                  2 Germany
```

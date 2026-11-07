# Return classified NUTS data

`nuts_get_data()` returns the classified data after running
[`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md).

## Usage

``` r
nuts_get_data(data)
```

## Arguments

- data:

  A nuts.classified object returned by
  [`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md).

## Value

A tibble containing the original data with the classified NUTS version,
level, and country.

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

nuts_get_data(classified)
#> # A tibble: 36 × 5
#>    from_code from_version from_level country values
#>    <chr>     <chr>             <dbl> <chr>    <dbl>
#>  1 DE11      2006                  2 Germany  11320
#>  2 DE12      2006                  2 Germany   3710
#>  3 DE13      2006                  2 Germany   9710
#>  4 DE14      2006                  2 Germany  11220
#>  5 DE21      2006                  2 Germany  22760
#>  6 DE22      2006                  2 Germany  16390
#>  7 DE23      2006                  2 Germany  12500
#>  8 DE24      2006                  2 Germany   8440
#>  9 DE25      2006                  2 Germany  10380
#> 10 DE26      2006                  2 Germany   6150
#> # ℹ 26 more rows
```

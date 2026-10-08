# Convert between NUTS versions

`nuts_convert_version()` transforms regional NUTS data between NUTS
versions.

## Usage

``` r
nuts_convert_version(
  data,
  to_version,
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

- to_version:

  String with desired NUTS version the function should convert to.
  Possible versions: `'2006'`, `'2010'`, `'2013'`, `'2016'` or `'2021'`

- variables:

  Named character specifying variable names and variable type
  (`'absolute'` or `'relative'`) e.g. `c('var_name' = 'absolute')`

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

A tibble containing NUTS codes, converted variable values, and possibly
grouping variables.

## Details

Console messages can be controlled with
`rlang::local_options(nuts.verbose = "quiet")` to silence messages and
`nuts.verbose = "verbose"` to switch messages back on.

## Examples

``` r
library(dplyr)

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

# Convert NUTS 2 codes in Germany from 2006 to 2021 version
manure %>%
  filter(nchar(geo) == 4) %>%
  filter(indic_ag == 'I07A_EQ_Y') %>%
  filter(grepl('^DE', geo)) %>%
  filter(time == 2003) %>%
  select(-indic_ag, -time) %>%
  # Data now only varies at the NUTS code level
  nuts_classify(nuts_code = "geo") %>%
  nuts_convert_version(to_version = '2021',
                       weight = 'pop21',
                       variables = c('values' = 'absolute'))
#> 
#> ── Classifying version of NUTS codes ───────────────────────────────────────────
#> Within groups defined by country:
#> ! These NUTS codes cannot be identified or classified: DEZZ.
#> ✔ Unique NUTS version classified.
#> ✖ Missing NUTS codes detected. See the tibble 'missing_data' in the output.
#> 
#> ── Converting version of NUTS codes ────────────────────────────────────────────
#> Within groups defined by country:
#> ℹ Converting NUTS codes in 1 version 2006 to version 2021.
#> ✖ These NUTS codes cannot be converted and are dropped: DEZZ.
#> ✔ Version is unique.
#> ✖ Missing NUTS codes in data. No values are calculated for regions associated
#>   with missing NUTS codes. Ensure that the input data is complete.
#> # A tibble: 38 × 4
#>    to_code to_version country values
#>    <chr>   <chr>      <chr>    <dbl>
#>  1 DE11    2021       Germany  11320
#>  2 DE12    2021       Germany   3710
#>  3 DE13    2021       Germany   9710
#>  4 DE14    2021       Germany  11220
#>  5 DE21    2021       Germany  22760
#>  6 DE22    2021       Germany  16390
#>  7 DE23    2021       Germany  12500
#>  8 DE24    2021       Germany   8440
#>  9 DE25    2021       Germany  10380
#> 10 DE26    2021       Germany   6150
#> # ℹ 28 more rows


# Convert NUTS 3 codes by country x year, classifying version first
manure %>%
  filter(nchar(geo) == 5) %>%
  filter(indic_ag == 'I07A_EQ_Y') %>%
  select(-indic_ag) %>%
  # Data now varies at the year x NUTS code level
  nuts_classify(nuts_code = 'geo', group_vars = c('time')) %>%
  nuts_convert_version(to_version = '2021',
                       weight = 'pop21',
                       variables = c('values' = 'absolute'))
#> 
#> ── Classifying version of NUTS codes ───────────────────────────────────────────
#> Within groups defined by country and time:
#> ! These NUTS codes cannot be identified or classified: ME000 and NOZZZ.
#> ✔ Unique NUTS version classified.
#> ✖ Missing NUTS codes detected. See the tibble 'missing_data' in the output.
#> 
#> ── Converting version of NUTS codes ────────────────────────────────────────────
#> Within groups defined by country and time:
#> ℹ Converting NUTS codes in 6 versions 2024, 2016, 2010, 2006, 2013, and 2021 to
#>   version 2021.
#> ✖ These NUTS codes cannot be converted and are dropped: ME000 and NOZZZ.
#> ✔ Version is unique.
#> ✖ Missing NUTS codes in data. No values are calculated for regions associated
#>   with missing NUTS codes. Ensure that the input data is complete.
#> # A tibble: 2,296 × 5
#>    to_code to_version country  time values
#>    <chr>   <chr>      <chr>   <dbl>  <dbl>
#>  1 AT111   2021       Austria  2010    276
#>  2 AT112   2021       Austria  2010    482
#>  3 AT113   2021       Austria  2010   1422
#>  4 AT121   2021       Austria  2010   6889
#>  5 AT122   2021       Austria  2010   3133
#>  6 AT123   2021       Austria  2010   2027
#>  7 AT124   2021       Austria  2010   5870
#>  8 AT125   2021       Austria  2010    905
#>  9 AT126   2021       Austria  2010   1103
#> 10 AT127   2021       Austria  2010    506
#> # ℹ 2,286 more rows

```

# Classify version of NUTS codes

`nuts_classify()` can identify the NUTS version year and level from a
variable containing NUTS codes.

## Usage

``` r
nuts_classify(
  data,
  nuts_code,
  group_vars = NULL,
  ties = c("most_recent", "oldest")
)
```

## Arguments

- data:

  A data frame or tibble that contains a variable with NUTS `1`, `2` or
  `3` codes and possibly other variables. NUTS codes must be of the same
  level and need to be unique, unless additional grouping variables are
  specified. No duplicate NUTS codes within groups allowed.

- nuts_code:

  Variable name containing NUTS codes

- group_vars:

  Variable name(s) for classification within groups. `nuts_classify()`
  always computes overlap within country. Hence, country variables
  should not be specified. `NULL` by default.

- ties:

  Picks `'most_recent'` or `'oldest'` version when overlap is identical
  across multiple NUTS versions. `'most_recent'` by default.

## Value

A list of three tibbles. The first tibble contains the original data
with the classified NUTS version, level, and country. The second tibble
lists the group-specific overlap with each NUTS version. The third
tibble shows missing NUTS codes for each group.

The output can be passed to
[`nuts_convert_version()`](https://docs.ropensci.org/nuts/reference/nuts_convert_version.md)
to convert data across NUTS versions and
[`nuts_aggregate()`](https://docs.ropensci.org/nuts/reference/nuts_aggregate.md)
to aggregate across NUTS levels.

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

# Classify version of NUTS 2 codes in Germany
manure %>%
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
#> $data
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
#> 
#> $versions_data
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
#> 
#> $missing_data
#> # A tibble: 4 × 4
#>   from_code from_version from_level country
#>   <chr>     <chr>             <dbl> <chr>  
#> 1 DE30      2006                  2 Germany
#> 2 DE50      2006                  2 Germany
#> 3 DE60      2006                  2 Germany
#> 4 DEC0      2006                  2 Germany
#> 
#> attr(,"groups")
#> [1] "country"
#> attr(,"class")
#> [1] "nuts.classified" "list"           

# Classify version of NUTS 3 codes within country and year
manure %>%
  filter(nchar(geo) == 5) %>%
  filter(indic_ag == 'I07A_EQ_Y') %>%
  select(-indic_ag) %>%
  # Data varies at the year x country x NUTS code level. The country grouping
  # is always used by default.
  nuts_classify(nuts_code = 'geo', group_vars = 'time')
#> 
#> ── Classifying version of NUTS codes ───────────────────────────────────────────
#> Within groups defined by country and time:
#> ! These NUTS codes cannot be identified or classified: ME000 and NOZZZ.
#> ✔ Unique NUTS version classified.
#> ✖ Missing NUTS codes detected. See the tibble 'missing_data' in the output.
#> $data
#> # A tibble: 1,902 × 6
#>    from_code from_version from_level country  time values
#>    <chr>     <chr>             <dbl> <chr>   <dbl>  <dbl>
#>  1 AT111     2024                  3 Austria  2010    276
#>  2 AT112     2024                  3 Austria  2010    482
#>  3 AT113     2024                  3 Austria  2010   1422
#>  4 AT121     2024                  3 Austria  2010   6889
#>  5 AT122     2024                  3 Austria  2010   3133
#>  6 AT123     2024                  3 Austria  2010   2027
#>  7 AT124     2024                  3 Austria  2010   5870
#>  8 AT125     2024                  3 Austria  2010    905
#>  9 AT126     2024                  3 Austria  2010   1103
#> 10 AT127     2024                  3 Austria  2010    506
#> # ℹ 1,892 more rows
#> 
#> $versions_data
#> # A tibble: 370 × 4
#>    from_version country  time overlap_perc
#>    <chr>        <chr>   <dbl>        <dbl>
#>  1 2024         Austria  2010          100
#>  2 2021         Austria  2010          100
#>  3 2016         Austria  2010          100
#>  4 2013         Austria  2010          100
#>  5 2010         Austria  2010          100
#>  6 2006         Austria  2010          100
#>  7 2016         Belgium  2000          100
#>  8 2013         Belgium  2000          100
#>  9 2010         Belgium  2000          100
#> 10 2006         Belgium  2000          100
#> # ℹ 360 more rows
#> 
#> $missing_data
#> # A tibble: 324 × 5
#>    from_code from_version from_level country  time
#>    <chr>     <chr>             <dbl> <chr>   <dbl>
#>  1 CZ010     2024                  3 Czechia  2003
#>  2 DK011     2024                  3 Denmark  2003
#>  3 DK012     2024                  3 Denmark  2003
#>  4 EE001     2016                  3 Estonia  2003
#>  5 EE007     2016                  3 Estonia  2003
#>  6 EL111     2010                  3 Greece   2003
#>  7 EL112     2010                  3 Greece   2003
#>  8 EL113     2010                  3 Greece   2003
#>  9 EL114     2010                  3 Greece   2003
#> 10 EL115     2010                  3 Greece   2003
#> # ℹ 314 more rows
#> 
#> attr(,"groups")
#> [1] "country" "time"   
#> attr(,"class")
#> [1] "nuts.classified" "list"           

```

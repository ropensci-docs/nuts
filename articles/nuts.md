# nuts: Convert European Regional Data in R

## Key Features

![Package logo of a squirrel holdling a walnut colored with the flag of
Europe](logo.png)

- Efficient **offline conversion** of European regional data.

- Conversion between five NUTS **versions**: 2006, 2010, 2013, 2016,
  2021.

- Conversion between three regional **levels**: NUTS-1, NUTS-2, NUTS-3.

- Ability to convert **multiple** NUTS versions at once when e.g. NUTS
  versions differ across countries and years. This scenario is common
  when working with data sourced from EUROSTAT.

- (Dasymetric) **Spatial interpolation** based on six **weights**
  (regional area size, 2011 and 2021 population size, 2012 and 2018
  artificial surfaces, and 2021 residential built-up volume) built from
  granular \[100m x 100m\] geodata by the European Commission’s [Joint
  Research Center
  (JRC)](https://urban.jrc.ec.europa.eu/tools/nuts-converter).

## NUTS Codes

The Nomenclature of Territorial Units for Statistics (NUTS) is a geocode
standard for referencing the administrative divisions of European
countries. A NUTS code starts with a two-letter combination indicating
the country.[^1] The administrative subdivisions, or **levels**, are
referred to with an additional number or a capital letter (NUTS-1). A
second (NUTS-2) or third (NUTS-3) subdivision level is referred to with
another digit each.

For example, the German district *Northern Saxony* (*Nordsachsen*) is
located within the region *Leipzig* and the federate state Saxony.

- NUTS-1: States
  - DED: Saxony
- NUTS-2: States/Government Regions
  - DED5: Leipzig
- NUTS-3: Districts
  - DED53: Northern Saxony

Since administrative boundaries in Europe change for demographic,
economic, political or other reasons, there are six different
**versions** of the NUTS Nomenclature (2006, 2010, 2013, 2016, 2021, and
2024).

### Spatial interpolation in a nutshell

When administrative units are restructured, regional data measured
within old boundaries can be converted to the new boundaries under
reasonable assumptions. The main task of this package is to use
(**dasymetric**) **spatial interpolation** to accomplish this.

Let’s take the example of the German state Saxony in the figures below.
Here, the NUTS-2 regions *Leipzig* (`DED3` → `DED5`) and *Chemnitz*
(`DED1` → `DED4`) were reorganized. We are interested in the number of
manure storage facilities in 2003 provided by
[EUROSTAT](https://ec.europa.eu/eurostat/databrowser/view/PAT_EP_RTOT/default/table)
based on the 2006 NUTS version. A part of *Leipzig* was reassigned to
*Chemnitz* (center plot), prompting us to recalculate the number of
storage facilities in the 2010 version (right plot).

A simple approach is to redistribute manure storage facilities
proportional to the transferred area, assuming equal distribution of
manure storages across space. In a dasymetric approach, we could make
use of built-up area, assuming that manure deposits are more likely to
be found close to residential areas and economic sites. In our example,
*Leipzig* lost about 7.7% (\\\frac{5574}{72360}\\) of its built-up area.
We re-calculate the number of manure storage facilities by computing
7.7% of *Leipzig’s* manure storages \\\frac{5574}{72360} \* 700 = 54\\,
subtracting them from Leipzig and adding them to *Chemnitz*.

See the Section [*Spatial interpolation in detail*](#method) for an
in-depth description of the weighting procedure.

![Maps of NUTS 3 regions Chemnitz and Leipzig in NUTS version 2003,
between 2003 and 2006 and 2006. They visualize the example in the text
in which Chemnitz contributes a part of its area to
Leipzig.](nuts_files/figure-html/unnamed-chunk-6-1.png)

Holdings with Manure Storage Facilities; BU = Built-up area in square
meters; Sources:
[Shapefiles](https://ec.europa.eu/eurostat/web/gisco/geodata/reference-data/administrative-units-statistical-units/nuts)
and
[data](https://ec.europa.eu/eurostat/databrowser/view/PAT_EP_RTOT/default/table)
are from EUROSTAT; Created using the
[sf](https://r-spatial.github.io/sf/) package.

## Usage

The package comes with three main functions:

- [`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md)
  detects the NUTS version(s) and level(s) of a data set. Its output can
  be directly fed into the two other functions.

- [`nuts_convert_version()`](https://docs.ropensci.org/nuts/reference/nuts_convert_version.md)
  converts your data to a desired NUTS version (2006, 2010, 2013, 2016,
  2021 and 2024). This transformation works in any direction.

- [`nuts_aggregate()`](https://docs.ropensci.org/nuts/reference/nuts_aggregate.md)
  aggregates data to some upper-level NUTS code, i.e., it transforms
  NUTS-3 data to the NUTS-2 or NUTS-1 level (but not vice versa).

### Workflow

The conversion can only be conducted after classifying the NUTS
version(s) and level(s) of your data using the function
[`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md).
This step ensures the validity and completeness of your NUTS codes
before proceeding with the conversion.

![Flow diagram that shows that conversion functions are run after
classification.](flow.png)

Sequential workflow to convert regional NUTS data

### Identifying NUTS version and level

The
[`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md)
function’s main purpose is to find the most suitable NUTS **version**
and to identify the **level** of the data set. Below, you see an example
using patent application data (per one million inhabitants) for Norway
in 2012 at the NUTS-2 level. This data is again provided by
[EUROSTAT](https://ec.europa.eu/eurostat/databrowser/view/PAT_EP_RTOT/default/table?lang=en).

``` r

# Load packages
library(nuts)
library(dplyr)
library(stringr)

# Loading and subsetting Eurostat data
data(patents, package = "nuts")

pat_n2 <- patents %>% 
  filter(nchar(geo) == 4) # NUTS-2 values

pat_n2_mhab_12_no <- pat_n2 %>%
  filter(unit == "P_MHAB") %>% # Patents per one million inhabitants
  filter(time == 2012) %>% # 2012
  filter(str_detect(geo, "^NO")) %>%  # Norway
  dplyr::select(-unit)

# Classifying the Data
pat_classified <- nuts_classify(
  data = pat_n2_mhab_12_no,
  nuts_code = "geo"
  )
```

    ## 

    ## ── Classifying version of NUTS codes ─────────────────────────────────

    ## Within groups defined by country:

    ## ✔ All NUTS codes can be identified and classified.

    ## ✔ Unique NUTS version classified.

    ## ✔ No missing NUTS codes.

The function returns a list with three items. These items can be called
directly from the output object (`data$...`) or retrieved using the
three helper functions
[`nuts_get_data()`](https://docs.ropensci.org/nuts/reference/nuts_get_data.md),
[`nuts_get_version()`](https://docs.ropensci.org/nuts/reference/nuts_get_version.md),
and
[`nuts_get_missing()`](https://docs.ropensci.org/nuts/reference/nuts_get_missing.md).

1.  The first item gives the **original data set** augmented with the
    columns `from_version`, `from_level`, and `country`, indicating the
    NUTS version that best suits the data. All functions of the package
    always group NUTS codes across **country names** which are
    automatically generated from the provided NUTS codes.

Below, you see that all data entries correspond to the 2016 NUTS
version.

``` r

# pat_classified$data # Call list item directly or...
nuts_get_data(pat_classified) # ...use helper function
```

    ## # A tibble: 7 × 6
    ##   from_code from_version from_level country  time values
    ##   <chr>     <chr>             <dbl> <chr>   <dbl>  <dbl>
    ## 1 NO01      2016                  2 Norway   2012  125. 
    ## 2 NO02      2016                  2 Norway   2012   13.2
    ## 3 NO03      2016                  2 Norway   2012   57.4
    ## 4 NO04      2016                  2 Norway   2012  110. 
    ## 5 NO05      2016                  2 Norway   2012   48.9
    ## 6 NO06      2016                  2 Norway   2012  145. 
    ## 7 NO07      2016                  2 Norway   2012   16.5

2.  The second item provides an overview of the share of matching NUTS
    codes for each of the five existing NUTS versions. The **overlap**
    is computed within country and possibly additional groups (if
    provided via the `group_vars` argument).

``` r

# pat_classified$versions_data # Call list item directly or...
nuts_get_version(pat_classified) # ...use helper function
```

    ## # A tibble: 6 × 3
    ##   from_version country overlap_perc
    ##   <chr>        <chr>          <dbl>
    ## 1 2016         Norway         100  
    ## 2 2013         Norway         100  
    ## 3 2010         Norway         100  
    ## 4 2006         Norway         100  
    ## 5 2024         Norway          42.9
    ## 6 2021         Norway          42.9

3.  The third item gives all NUTS codes that are **missing** across
    groups. Such missing codes might lead to conversion errors and are,
    by default, omitted from all conversion procedures. In our example,
    no NUTS codes are missing.

``` r

# pat_classified$missing_data # Call list item directly or...
nuts_get_missing(pat_classified) # ...use helper function
```

    ## # A tibble: 0 × 4
    ## # ℹ 4 variables: from_code <chr>, from_version <chr>,
    ## #   from_level <dbl>, country <chr>

### Converting data between NUTS versions

Once the NUTS version and level of the original data are identified, you
can easily **convert** the data to any other **NUTS version**. Here is
an example of transforming the 2013 Norwegian data to the 2021 NUTS
version. Between 2016 and 2021, the number of NUTS-2 regions in Norway
decreased by one as the borders of six regions were transformed. The
maps below show the affected regions. We provide the classified NUTS
data, specify the target NUTS version for data transformation, and
supply the variable containing the values to be interpolated. It is
important to indicate the **variable type** in the named input-vector
since the interpolation approaches differ for [absolute and relative
values](https://urban.jrc.ec.europa.eu/static/sites/nutsconverter/2022_08_04_NUTS_converter.pdf).

``` r

# Converting Data to 2021 NUTS version
pat_converted <- nuts_convert_version(
  data = pat_classified,
  to_version = "2021",
  variables = c("values" = "relative")
)
```

    ## 

    ## ── Converting version of NUTS codes ──────────────────────────────────

    ## Within groups defined by country:

    ## ℹ Converting NUTS codes in 1 version 2016 to version 2021.

    ## ✔ All NUTS codes can be converted.

    ## ✔ Version is unique.

    ## ✔ No missing NUTS codes.

![Two maps of patents per 1M habitants in Norwegian NUTS 2 regions in
NUTS version 2016 and converted to NUTS version
2021](nuts_files/figure-html/unnamed-chunk-14-1.png)

Converting patent data between versions; Sources:
[Shapefiles](https://ec.europa.eu/eurostat/web/gisco/geodata/reference-data/administrative-units-statistical-units/nuts)
and
[data](https://ec.europa.eu/eurostat/databrowser/view/PAT_EP_RTOT/default/table?lang=en)
are from EUROSTAT; Created using the
[sf](https://r-spatial.github.io/sf/) package.

The output below displays the corresponding data frames based on the
original and converted NUTS codes. The original data set comprises of
seven observations, whereas the converted data set contains six. The
regions `NO01`, `NO03`, `NO04`, and `NO05` are lost, while `NO08`,
`NO09`, and `NO0A` are now listed.

``` r

pat_n2_mhab_12_no
```

    ## # A tibble: 7 × 3
    ##   geo    time values
    ##   <chr> <dbl>  <dbl>
    ## 1 NO01   2012  125. 
    ## 2 NO02   2012   13.2
    ## 3 NO03   2012   57.4
    ## 4 NO04   2012  110. 
    ## 5 NO05   2012   48.9
    ## 6 NO06   2012  145. 
    ## 7 NO07   2012   16.5

``` r

pat_converted
```

    ## # A tibble: 6 × 4
    ##   to_code to_version country values
    ##   <chr>   <chr>      <chr>    <dbl>
    ## 1 NO02    2021       Norway    13.2
    ## 2 NO06    2021       Norway   143. 
    ## 3 NO07    2021       Norway    16.5
    ## 4 NO08    2021       Norway    71.0
    ## 5 NO09    2021       Norway    83.0
    ## 6 NO0A    2021       Norway    58.9

#### Converting multiple variables simultaneously

You can also convert **multiple variables** at once. Below, we add the
number of patent applications per 1000 inhabitants as a second variable:

``` r

# Converting Multiple Variables
pat_n2_mhab_12_no %>%
  mutate(values_per_thous = values * 1000) %>%
  nuts_classify(
    data = .,
    nuts_code = "geo"
    ) %>%
  nuts_convert_version(
    data = .,
    to_version = "2021",
    variables = c("values" = "relative",
                  "values_per_thous" = "relative")
  )
```

    ## 

    ## ── Classifying version of NUTS codes ─────────────────────────────────

    ## Within groups defined by country:

    ## ✔ All NUTS codes can be identified and classified.

    ## ✔ Unique NUTS version classified.

    ## ✔ No missing NUTS codes.

    ## 

    ## ── Converting version of NUTS codes ──────────────────────────────────

    ## Within groups defined by country:

    ## ℹ Converting NUTS codes in 1 version 2016 to version 2021.

    ## ✔ All NUTS codes can be converted.

    ## ✔ Version is unique.

    ## ✔ No missing NUTS codes.

    ## # A tibble: 6 × 5
    ##   to_code to_version country values values_per_thous
    ##   <chr>   <chr>      <chr>    <dbl>            <dbl>
    ## 1 NO02    2021       Norway    13.2           13239 
    ## 2 NO06    2021       Norway   143.           143106.
    ## 3 NO07    2021       Norway    16.5           16463 
    ## 4 NO08    2021       Norway    71.0           71038.
    ## 5 NO09    2021       Norway    83.0           82964.
    ## 6 NO0A    2021       Norway    58.9           58904.

#### Converting grouped data

Longitudinal regional data, as commonly supplied by EUROSTAT, often
comes with varying NUTS versions across countries and years (and other
dimensions). It is possible to harmonize data across such **groups**
with the `group_vars` argument in
[`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md).
Below, we transform data within country and year groups for Sweden,
Slovenia, and Croatia to the 2021 NUTS version.

``` r

# Classifying grouped data (time)
pat_n2_mhab_sesihr <- pat_n2 %>%
    filter(unit == "P_MHAB") %>%
    filter(str_detect(geo, "^SE|^SI|^HR"))

pat_classified <- nuts_classify(nuts_code = "geo", data = pat_n2_mhab_sesihr,
    group_vars = "time")
```

    ## 

    ## ── Classifying version of NUTS codes ─────────────────────────────────

    ## Within groups defined by country and time:

    ## ✔ All NUTS codes can be identified and classified.

    ## ✔ Unique NUTS version classified.

    ## ✔ No missing NUTS codes.

Note that the detected best-fitting NUTS versions differ across
countries:

``` r

nuts_get_data(pat_classified) %>%
    group_by(country, from_version) %>%
    tally()
```

    ## # A tibble: 3 × 3
    ## # Groups:   country [3]
    ##   country  from_version     n
    ##   <chr>    <chr>        <int>
    ## 1 Croatia  2016            24
    ## 2 Slovenia 2010            26
    ## 3 Sweden   2024           104

The grouping is stored and passed on to the conversion function:

``` r

# Converting grouped data (Time)
pat_converted <- nuts_convert_version(
  data = pat_classified,
  to_version = "2021",
  variables = c("values" = "relative")
)
```

    ## 

    ## ── Converting version of NUTS codes ──────────────────────────────────

    ## Within groups defined by country and time:

    ## ℹ Converting NUTS codes in 3 versions 2016, 2024, and 2010 to version
    ##   2021.

    ## ✔ All NUTS codes can be converted.

    ## ✔ Version is unique.

    ## ✔ No missing NUTS codes.

Conveniently, the group argument can also be used to transform higher
dimensional data. Below, we include two indicators for patent
applications to convert data that varies at the
indicator-year-country-NUTS code level.

``` r

# Classifying and converting multi-group data
pat_n2_mhabmact_12_sesihr <- pat_n2 %>%
  filter(unit %in% c("P_MHAB", "P_MACT")) %>%
  filter(str_detect(geo, "^SE|^SI|^HR"))

pat_converted <- pat_n2_mhabmact_12_sesihr %>%
  nuts_classify(
    data = .,
    nuts_code = "geo",
    group_vars = c("time", "unit")
  ) %>%
  nuts_convert_version(
    data = .,
    to_version = "2021",
    variables = c("values" = "relative")
  )
```

    ## 

    ## ── Classifying version of NUTS codes ─────────────────────────────────

    ## Within groups defined by country, time, and unit:

    ## ✔ All NUTS codes can be identified and classified.

    ## ✔ Unique NUTS version classified.

    ## ✔ No missing NUTS codes.

    ## 

    ## ── Converting version of NUTS codes ──────────────────────────────────

    ## Within groups defined by country, time, and unit:

    ## ℹ Converting NUTS codes in 3 versions 2016, 2024, and 2010 to version
    ##   2021.

    ## ✔ All NUTS codes can be converted.

    ## ✔ Version is unique.

    ## ✔ No missing NUTS codes.

### Converting data between NUTS levels

The
[`nuts_aggregate()`](https://docs.ropensci.org/nuts/reference/nuts_aggregate.md)
function facilitates the **aggregation** of data from lower NUTS
**levels** to higher ones using spatial weights. This enables users to
summarize variables upward from the NUTS-3 level to NUTS-2 or NUTS-1
levels. It is important to note that this function does not support
disaggregation since this comes with strong assumptions about the
spatial distribution of a variable’s values.

In the following example, we illustrate how to aggregate the total
number of patent applications in Sweden from NUTS-3 to higher levels.
The functions below return a warning concerning non-identifiable NUTS
codes. See [*Non-identified NUTS codes*](#nuts_not_identified) for
further information.

``` r

data("patents", package = "nuts")
# Aggregating data from NUTS-3 to NUTS-2 and NUTS-1
pat_n3 <- patents %>% 
  filter(nchar(geo) == 5)

pat_n3_nr_12_se <- pat_n3 %>%
  filter(unit %in% c("NR")) %>%
  filter(time == 2012) %>%
  filter(str_detect(geo, "^SE"))

pat_classified <- nuts_classify(
  data = pat_n3_nr_12_se,
  nuts_code = "geo"
  )
```

    ## 

    ## ── Classifying version of NUTS codes ─────────────────────────────────

    ## Within groups defined by country:

    ## ! These NUTS codes cannot be identified or classified: SEXXX and
    ##   SEZZZ.

    ## ✔ Unique NUTS version classified.

    ## ✔ No missing NUTS codes.

``` r

pat_level2 <- nuts_aggregate(
  data = pat_classified,
  to_level = 2,
  variables = c("values" = "absolute")
)
```

    ## 

    ## ── Aggregating level of NUTS codes ───────────────────────────────────

    ## Within groups defined by country:

    ## ℹ Aggregate from NUTS regional level 3 to 2.

    ## ✖ These NUTS codes cannot be converted and are dropped: SEXXX and
    ##   SEZZZ.

    ## ✔ Version is unique.

    ## ✔ No missing NUTS codes.

``` r

pat_level1 <- nuts_aggregate(
  data = pat_classified,
  to_level = 1,
  variables = c("values" = "absolute")
)
```

    ## 

    ## ── Aggregating level of NUTS codes ───────────────────────────────────

    ## Within groups defined by country:

    ## ℹ Aggregate from NUTS regional level 3 to 1.

    ## ✖ These NUTS codes cannot be converted and are dropped: SEXXX and
    ##   SEZZZ.

    ## ✔ Version is unique.

    ## ✔ No missing NUTS codes.

![Three maps of Sweden with patent applications at the NUTS 3 level and
aggregated to NUTS level 2 and
1.](nuts_files/figure-html/unnamed-chunk-23-1.png)

Aggregating patents from NUTS 3 to NUTS 2 and NUTS 1; Sources:
[Shapefiles](https://ec.europa.eu/eurostat/web/gisco/geodata/reference-data/administrative-units-statistical-units/nuts)
and
[data](https://ec.europa.eu/eurostat/databrowser/view/PAT_EP_RTOT/default/table?lang=en)
are from EUROSTAT; Created using the
[sf](https://r-spatial.github.io/sf/) package.

### Inconsistent versions and levels

#### Non-identified NUTS codes

If the input data contains NUTS codes that cannot be identified in any
NUTS version, the output of `classify_nuts` lists all of these codes.
All conversion procedures
([`nuts_convert_version()`](https://docs.ropensci.org/nuts/reference/nuts_convert_version.md)
and
[`nuts_aggregate()`](https://docs.ropensci.org/nuts/reference/nuts_aggregate.md))
will work as expected while ignoring values for these regions.

The example below classifies 2012 patent data from Denmark. The original
EUROSTAT data contains the codes `DKZZZ` and `DKXXX`, which are not part
of the conversion matrices. Codes ending with the letter Z refer to
“[Extra-Regio](https://stat.gov.pl/en/regional-statistics/classification-of-territorial-units/classification-of-territorial-units-for-statistics-nuts/principles-for-creation-and-development-of-nuts-units/)”
territories. These codes collect statistics for territories that cannot
be attached to a certain region.[^2] Codes ending with the letter X
refer to observations with unknown regions.

``` r

pat_n3.nr.12.dk <- pat_n3 %>%
  filter(unit %in% c("NR")) %>%
  filter(time == 2012) %>%
  filter(str_detect(geo, "^DK"))

pat_classified <- nuts_classify(
  data = pat_n3.nr.12.dk, 
  nuts_code = "geo"
  )
```

    ## 

    ## ── Classifying version of NUTS codes ─────────────────────────────────

    ## Within groups defined by country:

    ## ! These NUTS codes cannot be identified or classified: DKXXX and
    ##   DKZZZ.

    ## ✔ Unique NUTS version classified.

    ## ✔ No missing NUTS codes.

#### Missing NUTS codes

[`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md)
also checks whether the NUTS codes provided are complete (or values of a
variable that the user wants to convert are missing for a region).
Missing values in the input data will, by default, result in missing
values for all affected transformed regions in the output data.

The example with Slovenia below illustrates this case.

``` r

pat_n3_nr_12_si <- pat_n3 %>%
  filter(unit %in% c("NR")) %>%
  filter(time == 2012) %>%
  filter(str_detect(geo, "^SI"))

pat_classified <- nuts_classify(
  data = pat_n3_nr_12_si, 
  nuts_code = "geo"
  )
```

    ## 

    ## ── Classifying version of NUTS codes ─────────────────────────────────

    ## Within groups defined by country:

    ## ! These NUTS codes cannot be identified or classified: SIXXX and
    ##   SIZZZ.

    ## ✔ Unique NUTS version classified.

    ## ✖ Missing NUTS codes detected. See the tibble 'missing_data' in the
    ##   output.

[`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md)
returns a warning that NUTS codes are missing in the input data. These
codes can be inspected by calling `nuts_get_missing(pat_classified)`.

``` r

nuts_get_missing(pat_classified)
```

    ## # A tibble: 2 × 4
    ##   from_code from_version from_level country 
    ##   <chr>     <chr>             <dbl> <chr>   
    ## 1 SI011     2010                  3 Slovenia
    ## 2 SI016     2010                  3 Slovenia

The resulting conversion returns three missing values as the source code
`SI011` transformed into `SI031` and the region `SI016` was split into
`SI036` and `SI037`.

``` r

nuts_convert_version(
  data = pat_classified, 
  to_version = "2021", 
  variables = c("values" = "absolute")
  ) %>% 
  filter(is.na(values))
```

    ## 

    ## ── Converting version of NUTS codes ──────────────────────────────────

    ## Within groups defined by country:

    ## ℹ Converting NUTS codes in 1 version 2010 to version 2021.

    ## ✖ These NUTS codes cannot be converted and are dropped: SIXXX and
    ##   SIZZZ.

    ## ✔ Version is unique.

    ## ✖ Missing NUTS codes in data. No values are calculated for regions
    ##   associated with missing NUTS codes. Ensure that the input data is
    ##   complete.

    ## # A tibble: 3 × 4
    ##   to_code to_version country  values
    ##   <chr>   <chr>      <chr>     <dbl>
    ## 1 SI031   2021       Slovenia     NA
    ## 2 SI036   2021       Slovenia     NA
    ## 3 SI037   2021       Slovenia     NA

Users have the option `missing_weights_pct` to investigate the
consequences of missing values in the converted data. Setting the
argument to `TRUE` returns a variable that indicates the percentage of
missing weights due to missing NUTS codes (or missing values in the
variable). The data frame below shows three regions that could not be
computed due to missing data. Values in region `SI036` could not be
computed since 97.9% of the weights are missing. Values for region
`SI037` are missing as well even though only 0.8% of its
population-weighted area is missing.

``` r

nuts_convert_version(
  data = pat_classified, 
  to_version = "2021", 
  weight = "pop21",
  variables = c("values" = "absolute"),
  missing_weights_pct = TRUE
  ) %>% 
  arrange(desc(values_na_w))
```

    ## Warning: There was 1 warning in `summarise()`.
    ## ℹ In argument: `across(matches("w"), sum, na.rm = TRUE)`.
    ## ℹ In group 1: `to_code = "SI031"`, `country = "Slovenia"`.
    ## Caused by warning:
    ## ! The `...` argument of `across()` is deprecated as of dplyr 1.1.0.
    ## Supply arguments directly to `.fns` through an anonymous function
    ## instead.
    ## 
    ##   # Previously
    ##   across(a:b, mean, na.rm = TRUE)
    ## 
    ##   # Now
    ##   across(a:b, \(x) mean(x, na.rm = TRUE))
    ## ℹ The deprecated feature was likely used in the nuts package.
    ##   Please report the issue at <https://github.com/ropensci/nuts>.

    ## 

    ## ── Converting version of NUTS codes ──────────────────────────────────

    ## Within groups defined by country:

    ## ℹ Converting NUTS codes in 1 version 2010 to version 2021.

    ## ✖ These NUTS codes cannot be converted and are dropped: SIXXX and
    ##   SIZZZ.

    ## ✔ Version is unique.

    ## ✖ Missing NUTS codes in data. No values are calculated for regions
    ##   associated with missing NUTS codes. Ensure that the input data is
    ##   complete.

    ## # A tibble: 12 × 5
    ##    to_code to_version country  values values_na_w
    ##    <chr>   <chr>      <chr>     <dbl>       <dbl>
    ##  1 SI031   2021       Slovenia  NA        100    
    ##  2 SI036   2021       Slovenia  NA         97.9  
    ##  3 SI037   2021       Slovenia  NA          0.870
    ##  4 SI032   2021       Slovenia   7.84       0    
    ##  5 SI033   2021       Slovenia   3.22       0    
    ##  6 SI034   2021       Slovenia  15.3        0    
    ##  7 SI035   2021       Slovenia   6.99       0    
    ##  8 SI038   2021       Slovenia   1.25       0    
    ##  9 SI041   2021       Slovenia  42.1        0    
    ## 10 SI042   2021       Slovenia   6.56       0    
    ## 11 SI043   2021       Slovenia   7.22       0    
    ## 12 SI044   2021       Slovenia   3.3        0

Using the the share of missing weights in combination with the option
`missing_rm`, the `nuts` package allows to recover some of the missing
regions approximately. We can achieve this by setting `missing_rm` to
`TRUE`, effectively assuming 0 for missing values. In the next step we
remove regions with a high share of missing weights from the output data
again. The data frame below shows that values for `SI037` could still be
used assuming 0 patents for 0.8% of the missing population-weighted area
to construct the region.

``` r

nuts_convert_version(
  data = pat_classified, 
  to_version = "2021", 
  weight = "pop21",
  variables = c("values" = "absolute"),
  missing_weights_pct = TRUE,
  missing_rm = TRUE
  ) %>% 
  filter(to_code %in% c("SI031", "SI036", "SI037")) %>% 
  mutate(values_imp = ifelse(values_na_w < 1, values, NA))
```

    ## 

    ## ── Converting version of NUTS codes ──────────────────────────────────

    ## Within groups defined by country:

    ## ℹ Converting NUTS codes in 1 version 2010 to version 2021.

    ## ✖ These NUTS codes cannot be converted and are dropped: SIXXX and
    ##   SIZZZ.

    ## ✔ Version is unique.

    ## ✖ Missing NUTS codes in data. No values are calculated for regions
    ##   associated with missing NUTS codes. Ensure that the input data is
    ##   complete.

    ## # A tibble: 3 × 6
    ##   to_code to_version country  values values_na_w values_imp
    ##   <chr>   <chr>      <chr>     <dbl>       <dbl>      <dbl>
    ## 1 SI031   2021       Slovenia  0         100          NA   
    ## 2 SI036   2021       Slovenia  0.325      97.9        NA   
    ## 3 SI037   2021       Slovenia  4.02        0.870       4.02

#### Multiple NUTS levels within groups

The package does not allow for the conversion of **multiple NUTS
levels** at once. The classification function will throw an error in
this case. The conversion needs to be conducted for every level
separately.

``` r

patents %>% 
  filter(nchar(geo) %in% c(4, 5), grepl("^EL", geo)) %>% 
  distinct(geo, .keep_all = T) %>% 
  nuts_classify(nuts_code = "geo", data = .)
```

    ## Error in `nuts_classify()`:
    ## ! Data contains NUTS codes from multiple levels (2 and 3).

#### Multiple NUTS versions within groups

Converting **multiple NUTS versions** within groups might lead to
erroneous spatial interpolations since overlaps between regions of
different versions are possible.

The example below illustrates this problem. We classify German and
Italian manure storage facility data from EUROSTAT without specifying
`group_vars`. Instead, we keep all unique NUTS codes to artificially
create a data set containing different NUTS versions.
[`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md)
returns a warning and by inspecting the identified versions, we see that
there are mixed versions within groups (the countries).

``` r

man_deit <- manure %>% 
  filter(grepl("^DE|^IT", geo)) %>%
  filter(nchar(geo) == 4, ) %>% 
  distinct(geo, .keep_all = T) %>% 
  nuts_classify(nuts_code = "geo", data = .)
```

    ## 

    ## ── Classifying version of NUTS codes ─────────────────────────────────

    ## Within groups defined by country:

    ## ! These NUTS codes cannot be identified or classified: DEZZ.

    ## ✖ Multiple NUTS versions classified. See the tibble 'versions_data'
    ##   in the output.

    ## ✖ Missing NUTS codes detected. See the tibble 'missing_data' in the
    ##   output.

``` r

nuts_get_data(man_deit) %>% 
  group_by(country, from_version) %>% 
  tally()
```

    ## # A tibble: 5 × 3
    ## # Groups:   country [3]
    ##   country from_version     n
    ##   <chr>   <chr>        <int>
    ## 1 Germany 2006            38
    ## 2 Germany 2024             3
    ## 3 Italy   2006             9
    ## 4 Italy   2024            21
    ## 5 NA      NA               1

When proceeding to the conversion with either
[`nuts_convert_version()`](https://docs.ropensci.org/nuts/reference/nuts_convert_version.md)
or
[`nuts_aggregate()`](https://docs.ropensci.org/nuts/reference/nuts_aggregate.md),
both functions will throw an error. For convenience, we added the option
`multiple_versions` that subsets the supplied data to the dominant
version within groups when specified with `most_frequent`. Hence, all
codes from other, non-dominant versions are discarded.

Once we convert this data set, all NUTS regions unrecognized according
to the 2006 (Germany) and 2021 (Italy) version are dropped
automatically.

``` r

man_deit_converted <- nuts_convert_version(
  data = man_deit,
  to_version = 2021,
  variables = c("values" = "relative"),
  multiple_versions = "most_frequent"
)
```

    ## 

    ## ── Converting version of NUTS codes ──────────────────────────────────

    ## Within groups defined by country:

    ## ℹ Converting NUTS codes in 2 versions 2006 and 2024 to version 2021.

    ## ✖ These NUTS codes cannot be converted and are dropped: DEZZ.

    ## ! Choosing most frequent version within group and dropping 12 rows.

    ## ✖ Missing NUTS codes in data. No values are calculated for regions
    ##   associated with missing NUTS codes. Ensure that the input data is
    ##   complete.

``` r

man_deit_converted %>% 
  group_by(country, to_version) %>% 
  tally()
```

    ## # A tibble: 2 × 3
    ## # Groups:   country [2]
    ##   country to_version     n
    ##   <chr>        <dbl> <int>
    ## 1 Germany       2021    38
    ## 2 Italy         2021    21

## Spatial interpolation in detail

This section describes the spatial interpolation procedure. We first
cover the logic of conversion tables and then explain the methods used
in the package for converting versions and levels.

### Changes in administrative boundaries

Below, Norwegian NUTS-2 regions for the versions 2016 and 2021 are
shown. All regions apart from Norway’s most Northern region have been
reorganized in this period.

![Two maps of Norwegian NUTS-2 regions in version 2016 and 2021. The
most Eastern and Southern regions have been affected most by
administrative
redistricting.](nuts_files/figure-html/unnamed-chunk-33-1.png)

Norwegian NUTS-2 regions with boundary changes; Sources:
[Shapefiles](https://ec.europa.eu/eurostat/web/gisco/geodata/reference-data/administrative-units-statistical-units/nuts)
from EUROSTAT; Created using the [sf](https://r-spatial.github.io/sf/)
package.

The changes between the two versions can be summarized as follows:

1.  Boundary changes of regions with **continued** NUTS codes

- `NO02` ceases a small area to the new `NO08`
- `NO06` makes small area gains from `NO05`

2.  Changes to regions with **discontinued** NUTS codes

- `NO01` is absorbed by `NO08`
- `NO03` is split up between `NO08` and `NO09`
- `NO04` divides into `NO0A` and `NO09`
- `NO05` largely becomes the new `NO0A`, and gives a small area to
  `NO06`

### Spatial interpolation and conversion tables

To keep track of these changes, the `nuts` package uses two data sets:

1.  Stocks: data(`all_nuts_codes`) contains **all historical NUTS
    codes** by NUTS version and country
2.  Flows: data(`cross_walks`) contains the **conversion tables**
    between NUTS versions

They are based on [data provided by the
JRC](https://urban.jrc.ec.europa.eu/tools/nuts-converter?lng=en). Both
data sets can also be used by the user manually to explore specific
conversion patterns more closely.

For Norway going from version 2016 to 2021 at NUTS level 2, the
`cross_walks` can be easily subset as follows:

``` r

no_walks <- cross_walks %>%
  filter(nchar(from_code) == 4,
         from_version == 2016,
         to_version == 2021,
         grepl("^NO", from_code))
```

Which results in the following conversion table:

| from_code | to_code | from_version | to_version | level | country | areaKm | pop21 | pop11 | artif_surf18 | artif_surf12 | bu_vol |
|:---|:---|:---|:---|---:|:---|---:|---:|---:|---:|---:|---:|
| NO01 | NO08 | 2016 | 2021 | 2 | Norway | 5364.9736 | 1327889 | 1131221.018 | 58104 | 55927 | 267187501 |
| NO02 | NO02 | 2016 | 2021 | 2 | Norway | 52072.3527 | 370291 | 362070.677 | 60625 | 54887 | 170850205 |
| NO02 | NO08 | 2016 | 2021 | 2 | Norway | 517.5545 | 15905 | 15018.954 | 1952 | 1813 | 4457002 |
| NO03 | NO08 | 2016 | 2021 | 2 | Norway | 19123.5099 | 593638 | 535560.504 | 66876 | 62509 | 196792528 |
| NO03 | NO09 | 2016 | 2021 | 2 | Norway | 17414.4384 | 416597 | 385648.848 | 50076 | 47799 | 127750896 |
| NO04 | NO09 | 2016 | 2021 | 2 | Norway | 16360.8666 | 302523 | 272016.250 | 44779 | 42346 | 100825532 |
| NO04 | NO0A | 2016 | 2021 | 2 | Norway | 9325.9681 | 471357 | 416975.372 | 39112 | 36432 | 144978619 |
| NO05 | NO06 | 2016 | 2021 | 2 | Norway | 931.7538 | 3503 | 3625.947 | 869 | 832 | 2272634 |
| NO05 | NO0A | 2016 | 2021 | 2 | Norway | 47902.0663 | 875103 | 790090.067 | 99951 | 94757 | 262414590 |
| NO06 | NO06 | 2016 | 2021 | 2 | Norway | 41028.9253 | 462448 | 417827.700 | 47291 | 43630 | 146314602 |
| NO07 | NO07 | 2016 | 2021 | 2 | Norway | 112452.8543 | 469239 | 437265.193 | 81907 | 79098 | 149055058 |

In addition to tracing the evolution of NUTS codes, the table contains
**flows** of area, population and artificial surfaces between regions
and versions. These flows were computed by the JRC with granular \[100m
x 100m\] geographic data. The `ggalluvial` plot below visualizes the
flows of area size between the NUTS-2 regions mapped above.

![The alluvial plot shows population flows from NUTS version 2016 to
2021.](nuts_files/figure-html/unnamed-chunk-36-1.png)

Alluvial plot illustrating area size flows; Created using the
[ggalluvial](https://corybrunson.github.io/ggalluvial/) package.

To illustrate the main idea, the map below showcases **population
densities** across NUTS-2 regions. As population is not uniformly
distributed across space, weighting regions dependent on their area size
comes with strong assumptions. For instance, region `NO01` in version
2016, that contains the city of Oslo, makes a relatively modest
geographical contribution to the new region `NO08`, but significantly
bolsters the population of the latter. Assuming that the variable to be
converted is correlated with population across space, the conversion can
thus be refined using population weights to account for flows between
different versions.

![Two maps of Southern Norway with very granular population density and
administrative boundaries of the 2016 and 2021 NUTS version. The region
with the capital Olso and its adjacent region are highlighted in version
2016 that both contribute to a larger single region in version
2021.](nuts_files/figure-html/unnamed-chunk-37-1.png)

Spatial distribution of population and boundary changes; Sources:
[Shapefiles](https://ec.europa.eu/eurostat/web/gisco/geodata/reference-data/administrative-units-statistical-units/nuts)
and [population
raster](https://ec.europa.eu/eurostat/web/gisco/geodata/reference-data/population-distribution-demography/geostat)
from EUROSTAT; Created using the [sf](https://r-spatial.github.io/sf/)
and the
[terra](https://rspatial.github.io/terra/reference/terra-package.html)
packages.

### Conversion methods

The following subsections describe the method used to convert absolute
and relative values between versions and levels.

#### Conversion of absolute values between versions

In this example, we transform **absolute** values, the number of patent
applications (`NR`) in Norway, from **version** 2016 to 2021, utilizing
spatial interpolation based on the population distribution in 2018.

The conversion employs the `cross_walks` table, which includes
population flow data (expressed in thousands) between two NUTS-2 regions
from the source version to the target version. The function joins the
variable of interest, `NR`, which varies across the departing NUTS-2
codes (`from_code`). The function initially calculates a **weight**
(`w`) equal to the population flow’s share of the total population in
the departing region in version 2016 (`from_code`):

| from_code | to_code | from_version | to_version |  NR | pop21 | w                      |
|:----------|:--------|:-------------|:-----------|----:|------:|:-----------------------|
| NO01      | NO08    | 2016         | 2021       | 146 |  1327 | 1327/(1327) = 1        |
| NO02      | NO02    | 2016         | 2021       |   5 |   370 | 370/(370 + 15) = 0.96  |
| NO02      | NO08    | 2016         | 2021       |   5 |    15 | 15/(370 + 15) = 0.04   |
| NO03      | NO08    | 2016         | 2021       |  54 |   593 | 593/(593 + 416) = 0.59 |
| NO03      | NO09    | 2016         | 2021       |  54 |   416 | 416/(593 + 416) = 0.41 |
| NO04      | NO09    | 2016         | 2021       |  80 |   302 | 302/(302 + 471) = 0.39 |
| NO04      | NO0A    | 2016         | 2021       |  80 |   471 | 471/(302 + 471) = 0.61 |
| NO05      | NO06    | 2016         | 2021       |  41 |     3 | 3/(3 + 875) = 0        |
| NO05      | NO0A    | 2016         | 2021       |  41 |   875 | 875/(3 + 875) = 1      |
| NO06      | NO06    | 2016         | 2021       |  62 |   462 | 462/(462) = 1          |
| NO07      | NO07    | 2016         | 2021       |   7 |   469 | 469/(469) = 1          |

To obtain the number of patent applications at the desired 2021 version,
the function summarizes the data for the new NUTS regions in version
2021 (`to_code`) by taking the **population-weighted sum** of all flows.

| to_code | to_version | NR                                      |
|:--------|:-----------|:----------------------------------------|
| NO02    | 2021       | 5 x 0.96 = 4.8                          |
| NO06    | 2021       | 41 x 0 + 62 x 1 = 62                    |
| NO07    | 2021       | 7 x 1 = 7                               |
| NO08    | 2021       | 146 x 1 + 5 x 0.04 + 54 x 0.59 = 178.06 |
| NO09    | 2021       | 54 x 0.41 + 80 x 0.39 = 53.34           |
| NO0A    | 2021       | 80 x 0.61 + 41 x 1 = 89.8               |

#### Conversion of relative values between versions

To convert **relative** values, such as the number of patent
applications per 1000 inhabitants,
[`nuts_convert_version()`](https://docs.ropensci.org/nuts/reference/nuts_convert_version.md)
departs again from the conversion table seen above. We focus on the
variable `P_MHAB`, patent applications per one million inhabitants. The
function summarizes these relative values by computing the **weighted
average** with respect to 2018 population flows.

| to_code | to_version | P_MHAB |
|:---|:---|:---|
| NO02 | 2021 | (370 x 13)/(370) = 13 |
| NO06 | 2021 | (3 x 48 + 462 x 145)/(3 + 462) = 144 |
| NO07 | 2021 | (469 x 16)/(469) = 16 |
| NO08 | 2021 | (1327 x 125 + 15 x 13 + 593 x 57)/(1327 + 15 + 593) = 103 |
| NO09 | 2021 | (416 x 57 + 302 x 110)/(416 + 302) = 79 |
| NO0A | 2021 | (471 x 110 + 875 x 48)/(471 + 875) = 70 |

#### Conversion of absolute values between NUTS levels

The function
[`nuts_aggregate()`](https://docs.ropensci.org/nuts/reference/nuts_aggregate.md)
aggregates from lower to higher order **levels**, e.g. from NUTS-3 to
NUTS-2. Since higher order regions are perfectly split into lower order
regions in the NUTS system, the function takes simply the sum of the
values in case of **absolute** variables.

#### Conversion of relative values between NUTS levels

**Relative** values are aggregated in
[`nuts_aggregate()`](https://docs.ropensci.org/nuts/reference/nuts_aggregate.md)
by computing the weighted mean of all lower order regional **levels**.
To convert, for example, the number of patent applications per one
million inhabitants from NUTS-3 to NUTS-2, the function adds the
population size in 2018.

| nuts_3 | nuts_2 | pop21 | P_MHAB |
|:-------|:-------|------:|-------:|
| NO011  | NO01   |   688 |    145 |
| NO012  | NO01   |   639 |    102 |
| NO021  | NO02   |   197 |      7 |
| NO022  | NO02   |   187 |     18 |
| NO031  | NO03   |   300 |     34 |
| NO032  | NO03   |   286 |     45 |
| NO033  | NO03   |   250 |    106 |
| NO034  | NO03   |   171 |     45 |
| NO041  | NO04   |   116 |     43 |
| NO042  | NO04   |   185 |     50 |
| NO043  | NO04   |   471 |    150 |
| NO051  | NO05   |   517 |     24 |
| NO052  | NO05   |   105 |     58 |
| NO053  | NO05   |   254 |     91 |
| NO061  | NO06   |   320 |    208 |
| NO062  | NO06   |   141 |      3 |
| NO071  | NO07   |   233 |     10 |
| NO072  | NO07   |   162 |     33 |

The number of patent applications at the NUTS-2 level is computed by the
**weighted average** using NUTS-3 population numbers.

| nuts_2 | P_MHAB |
|:---|:---|
| NO01 | (688 x 145 + 639 x 102)/(688 + 639) = 124 |
| NO02 | (197 x 7 + 187 x 18)/(197 + 187) = 12 |
| NO03 | (300 x 34 + 286 x 45 + 250 x 106 + 171 x 45)/(300 + 286 + 250 + 171) = 56 |
| NO04 | (116 x 43 + 185 x 50 + 471 x 150)/(116 + 185 + 471) = 109 |
| NO05 | (517 x 24 + 105 x 58 + 254 x 91)/(517 + 105 + 254) = 47 |
| NO06 | (320 x 208 + 141 x 3)/(320 + 141) = 145 |
| NO07 | (233 x 10 + 162 x 33)/(233 + 162) = 19 |

## Citation

Please support the development of open science and data by citing the
JRC and us in your work:

- Joint Research Centre (2022) NUTS converter.
  <https://urban.jrc.ec.europa.eu/tools/nuts-converter>

- Hennicke M, Krause W (2024). *nuts: Convert European Regional Data*.
  <doi:10.5281/zenodo.10573056>
  <https://doi.org/10.5281/zenodo.10573056>, R package version 1.1.0,
  <https://docs.ropensci.org/nuts/>.

Bibtex Users:

    @Manual{,
      title = {NUTS converter},
      author = {Joint Research Centre},
      year = {2022},
      url = {https://urban.jrc.ec.europa.eu/tools/nuts-converter},
    }

    @Manual{,
      title = {nuts: Convert European Regional Data},
      author = {Moritz Hennicke and Werner Krause},
      year = {2024},
      note = {R package version 1.1.0},
      url = {https://docs.ropensci.org/nuts/},
      doi = {10.5281/zenodo.10573056},
    }

[^1]: [Eurostat](https://ec.europa.eu/eurostat/web/nuts/history) In the
    case of Greece this code was changed from GR to EL in 2011.

[^2]: Such as air-space, territorial waters and the continental shelf,
    embassies, consulates, military bases and deposits of oil, natural
    gas, etc. in international waters.

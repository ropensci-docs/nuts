# Changelog

## nuts (development version)

- 2024 NUTS version now included
- 2018 population weight replaced for 2021 population weight
- 2021 residential built-up volume added as a weight
- Fixed version 2016 NUTS-3 codes for UK
- Fixed `object 'all_nuts_codes' not found` and
  `object 'cross_walks' not found` errors when calling
  [`nuts_classify()`](https://docs.ropensci.org/nuts/reference/nuts_classify.md),
  [`nuts_convert_version()`](https://docs.ropensci.org/nuts/reference/nuts_convert_version.md),
  and related functions via `nuts::` without first attaching the package
  with [`library(nuts)`](https://docs.ropensci.org/nuts/)
  ([\#9](https://github.com/ropensci/nuts/issues/9))
- Fixed `missing_data` incorrectly reporting NUTS codes as missing when
  they were present in the input but classified under a different NUTS
  version than the row being checked
  ([\#11](https://github.com/ropensci/nuts/issues/11))

## nuts 1.1.0

CRAN release: 2024-07-13

- Fixed bug in nuts_aggregate in the check for missing nuts codes

## nuts 1.0.0

CRAN release: 2024-03-22

- First release
- nuts is now a part of rOpenSci after
  [peer-review](https://github.com/ropensci/software-review/issues/623)!
- Functions were renamed to be consistent with rOpenSci’s style guide
- Verbosity control was added
- Repeated chunks of code were reorganized into functions
- Option for summarizing missing weights included
- User interface rewritten with `cli`
- Imports were further reduced
- Tests were reorganized and expanded
- Documentation was updated

## nuts 0.0.0.9000

- Development version launch

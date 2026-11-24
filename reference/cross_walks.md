# Conversion table provided by the Joint Research Center of the European Commission

The table contains population, area and surface flows between two NUTS
regions and different NUTS code classifications. NUTS regions are at
1st, 2nd and 3rd level. NUTS versions are 2006, 2010, 2013, 2016, 2021
and 2024.

## Usage

``` r
cross_walks
```

## Format

### `cross_walks`

A data frame with 66,877 rows and 12 columns:

- from_code:

  Departing NUTS code

- to_code:

  Desired NUTS code

- from_version:

  Departing NUTS version

- to_version:

  Desired NUTS version

- level:

  NUTS division level

- country:

  Country name

- areaKm:

  Area size flow

- pop21:

  2021 population flow

- pop11:

  2011 population flow

- artif_surf18:

  2018 artificial surfaces flow

- artif_surf12:

  2012 artificial surfaces flow

- bu_vol:

  2021 residential built-up volume flow

## Source

<https://urban.jrc.ec.europa.eu/tools/nuts-converter?lng=en#/>

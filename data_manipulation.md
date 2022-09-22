Data Manipulation with `dplyr`
================

Once you’ve imported data, you’re going to need to do some cleaning up.

## Overview

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.3.6      ✔ purrr   0.3.4 
    ## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
    ## ✔ tidyr   1.2.0      ✔ stringr 1.4.1 
    ## ✔ readr   2.1.2      ✔ forcats 0.5.2 
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
options(tibble.print_min = 3)

litters_data <- read_csv("./data/FAS_litters.csv")
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litters_data <- janitor::clean_names(litters_data)

pups_data <- read_csv("./data/FAS_pups.csv",
  col_types = "ciiiii")
pups_data <- janitor::clean_names(pups_data)
```

### `select`

You can specify the columns you want to keep by naming all of them:

``` r
select(litters_data, group, litter_number, gd0_weight, pups_born_alive)
```

    ## # A tibble: 49 × 4
    ##   group litter_number gd0_weight pups_born_alive
    ##   <chr> <chr>              <dbl>           <dbl>
    ## 1 Con7  #85                 19.7               3
    ## 2 Con7  #1/2/95/2           27                 8
    ## 3 Con7  #5/5/3/83/3-3       26                 6
    ## # … with 46 more rows

You can specify a range of columns to keep:

``` r
select(litters_data, group:gd_of_birth)
```

    ## # A tibble: 49 × 5
    ##   group litter_number gd0_weight gd18_weight gd_of_birth
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>
    ## 1 Con7  #85                 19.7        34.7          20
    ## 2 Con7  #1/2/95/2           27          42            19
    ## 3 Con7  #5/5/3/83/3-3       26          41.4          19
    ## # … with 46 more rows

You can also specify columns you’d like to remove:

``` r
select(litters_data, -pups_survive)
```

    ## # A tibble: 49 × 7
    ##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive pups_…¹
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>   <dbl>
    ## 1 Con7  #85                 19.7        34.7          20               3       4
    ## 2 Con7  #1/2/95/2           27          42            19               8       0
    ## 3 Con7  #5/5/3/83/3-3       26          41.4          19               6       0
    ## # … with 46 more rows, and abbreviated variable name ¹​pups_dead_birth

You can rename variables as part of this process:

``` r
select(litters_data, GROUP = group, LiTtEr_NuMbEr = litter_number)
```

    ## # A tibble: 49 × 2
    ##   GROUP LiTtEr_NuMbEr
    ##   <chr> <chr>        
    ## 1 Con7  #85          
    ## 2 Con7  #1/2/95/2    
    ## 3 Con7  #5/5/3/83/3-3
    ## # … with 46 more rows

If all you want to do is rename something, you can use `rename` instead
of `select`. This will rename the variables you care about, and keep
everything else:

``` r
rename(litters_data, GROUP = group, LiTtEr_NuMbEr = litter_number)
```

    ## # A tibble: 49 × 8
    ##   GROUP LiTtEr_NuMbEr gd0_weight gd18_weight gd_of_birth pups_…¹ pups_…² pups_…³
    ##   <chr> <chr>              <dbl>       <dbl>       <dbl>   <dbl>   <dbl>   <dbl>
    ## 1 Con7  #85                 19.7        34.7          20       3       4       3
    ## 2 Con7  #1/2/95/2           27          42            19       8       0       7
    ## 3 Con7  #5/5/3/83/3-3       26          41.4          19       6       0       5
    ## # … with 46 more rows, and abbreviated variable names ¹​pups_born_alive,
    ## #   ²​pups_dead_birth, ³​pups_survive

There are some handy helper functions for select; read about all of them
using ?select_helpers. I use starts_with(), ends_with(), and contains()
often, especially when there variables are named with suffixes or other
standard patterns:

``` r
select(litters_data, starts_with("gd"))
```

    ## # A tibble: 49 × 3
    ##   gd0_weight gd18_weight gd_of_birth
    ##        <dbl>       <dbl>       <dbl>
    ## 1       19.7        34.7          20
    ## 2       27          42            19
    ## 3       26          41.4          19
    ## # … with 46 more rows

I also frequently use is everything(), which is handy for reorganizing
columns without discarding anything:

``` r
select(litters_data, litter_number, pups_survive, everything())
```

    ## # A tibble: 49 × 8
    ##   litter_number pups_survive group gd0_weight gd18_wei…¹ gd_of…² pups_…³ pups_…⁴
    ##   <chr>                <dbl> <chr>      <dbl>      <dbl>   <dbl>   <dbl>   <dbl>
    ## 1 #85                      3 Con7        19.7       34.7      20       3       4
    ## 2 #1/2/95/2                7 Con7        27         42        19       8       0
    ## 3 #5/5/3/83/3-3            5 Con7        26         41.4      19       6       0
    ## # … with 46 more rows, and abbreviated variable names ¹​gd18_weight,
    ## #   ²​gd_of_birth, ³​pups_born_alive, ⁴​pups_dead_birth

`relocate` does a similar thing (and is sort of like rename in that it’s
handy but not critical):

``` r
relocate(litters_data, litter_number, pups_survive)
```

    ## # A tibble: 49 × 8
    ##   litter_number pups_survive group gd0_weight gd18_wei…¹ gd_of…² pups_…³ pups_…⁴
    ##   <chr>                <dbl> <chr>      <dbl>      <dbl>   <dbl>   <dbl>   <dbl>
    ## 1 #85                      3 Con7        19.7       34.7      20       3       4
    ## 2 #1/2/95/2                7 Con7        27         42        19       8       0
    ## 3 #5/5/3/83/3-3            5 Con7        26         41.4      19       6       0
    ## # … with 46 more rows, and abbreviated variable names ¹​gd18_weight,
    ## #   ²​gd_of_birth, ³​pups_born_alive, ⁴​pups_dead_birth

### `filter`

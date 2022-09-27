Tidy Data
================
2022-09-27

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
options(tibble.print_min = 5)
```

### Using `pivot_longer`

``` r
pulse_df <- haven::read_sas('data/public_pulse_data.sas7bdat') %>%
  janitor::clean_names()
```

Data shows ID, age at baseline, sex, and depression scores. Data is not
tidy. We want to `pivot_longer`, take those columnns, stack them on top
of each other, and create a new variable that indicates what visit we’re
at.

``` r
pulse_tidy_data <-
  pivot_longer(
    pulse_df, 
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    values_to = "bdi",
    names_prefix = "bdi_score_"
  )
```

`names_prefix` –\> drops whatever you list in quotes.

### Doing things in one swoop + `mutate`

``` r
pulse_df <- haven::read_sas('data/public_pulse_data.sas7bdat') %>%
  janitor::clean_names() %>% 
    pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    values_to = "bdi",
    names_prefix = "bdi_score_"
  ) %>% 
  mutate(
    visit = replace(visit, visit == "bl", "00m"),
    visit = factor(visit)
  ) %>% 
  arrange(id, visit)
```

### Learning Assessment: Litters Data

``` r
litters_df <- read_csv('data/FAS_litters.csv') %>% 
  janitor::clean_names()
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
litters_tidy_df <- litters_df %>% 
  select(litter_number, gd0_weight, gd18_weight) %>% 
  pivot_longer(
    c(gd0_weight, gd18_weight),
    names_to = "gd",
    values_to = "weight"
  ) %>% 
  mutate(
    gd = recode(gd, "gd0_weight" = 0, "gd18_weight" = 18))
```

Answer: Yes, this is relatively tidy. But this makes it harder to
compute weight change, so it may not be as useful. Need to think about
what you want to do. Sometimes this can be better, sometimes you’ll want
it long.

### `pivot_wider`

Creating tibble

``` r
analysis_results <- tibble(
  group = c('treatment', 'treatment', 'placebo', 'placebo'),
  time = c('pre', 'post', 'pre', 'post'),
  mean = c(4, 8, 3.5, 4)
)
```

What if we want to change this into wide format?

``` r
analysis_results_wide <- pivot_wider(
  analysis_results,
  names_from = 'time',
  values_from = 'mean'
)
```

`names_from` = Column names are coming from the variable `time`
`values_from` = Values come from the variable listed here that will move
into the new variable

### Lord of the Rings Exercise - Reading in all 3 tables

``` r
fellowship_ring <- readxl::read_excel('data/LotR_Words.xlsx', range = 'B3:D6') %>% 
  mutate(movie = 'fellowship_ring')

two_towers <- readxl::read_excel('data/LotR_Words.xlsx', range = 'F3:H6') %>% 
  mutate(movie = 'two_towers')

return_king <- readxl::read_excel('data/LotR_Words.xlsx', range = 'J3:L6') %>% 
  mutate(movie = 'return_king')

lotr_tidy <- bind_rows(fellowship_ring, two_towers, return_king) %>% 
  janitor::clean_names() %>% 
  pivot_longer(
    female:male,
    names_to = "gender",
    values_to = "words"
  ) %>% 
  mutate(race = str_to_lower(race))
```

### Joining Datasets + using `separate`

What we want to do:

-   Separate group (con, mod, and low, and the day they received the
    alcohol) \> two pieces of information in one column, and we want to
    separate them out

``` r
pups <-
  read_csv('data/FAS_pups.csv') %>% 
  janitor::clean_names() %>% 
  mutate(sex = recode(sex, `1` = "male", `2` = "female"))
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litter <-
  read_csv('data/FAS_litters.csv') %>% 
  janitor::clean_names() %>% 
  separate(group, into = c('dose', 'day_of_tx'), sep = 3) %>% 
    # sep = separate after the third character
  mutate(
    wt_gain = gd18_weight - gd0_weight,
    dose = str_to_lower(dose)
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Note: have to use ” `` ” (back quotes) to refer to a numeric value in a
numeric variable.

We’re going to do `left_join` because we want the pup data to be joined
to the mothers data \>\> we want to take the baby data and join mom’s to
that because our intent is to analyze the baby’s data.

``` r
fas_data <- 
  left_join(pups, litter)
```

    ## Joining, by = "litter_number"

### `anti-join`

This is a more elegant, tidyverse way of checking what you’re dropping.

``` r
anti_join(pups, litter)
```

    ## Joining, by = "litter_number"

    ## # A tibble: 9 × 6
    ##   litter_number sex    pd_ears pd_eyes pd_pivot pd_walk
    ##   <chr>         <chr>    <dbl>   <dbl>    <dbl>   <dbl>
    ## 1 #5/3/83/3-2   female       3      12       NA       8
    ## 2 #5/3/83/3-2   female       3      13       NA      10
    ## 3 #7/82/3-2     male         3      12        6       8
    ## 4 #7/82/3-2     male         4      13        5       8
    ## 5 #7/82/3-2     male         3      13        6       8
    ## 6 #7/82/3-2     female       3      13        6       8
    ## 7 #7/82/3-2     female       3      12        6       8
    ## 8 #7/82/3-2     female       3      12        6       8
    ## 9 #7/82/3-2     female       3      12        6       8

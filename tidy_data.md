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

### Lord of the Rings Exercise

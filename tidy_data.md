Tidy Data
================

This document will show how to tidy data.

## Pivot Longer

``` r
pulse_df = 
  read_sas("data/public_pulse_data.sas7bdat") |> 
  janitor::clean_names() |> 
  pivot_longer(
      cols = bdi_score_bl:bdi_score_12m,
      names_to = "visit",
      values_to = "bdi_score",
      names_prefix = "bdi_score_"   #strips all the prefix
  ) |> 
  mutate(
    visit = replace(visit, visit == "bl", "00m")
  ) |> 
  relocate(id, visit)
```

Do one more example.

``` r
litters_df = 
    read_csv("data/FAS_litters.csv", na = c("NA", ".", "")) |> 
    janitor::clean_names() |> 
    pivot_longer(
        cols = gd0_weight:gd18_weight, #name the columns you are interested in between gd0 and gd18 (this is in order, but can also use "starts with..." or "contains...")
        names_to = "gd_time",
        values_to = "weight"
    ) |> 
  mutate(
    gd_time = case_match(
      gd_time, #what variable to look at
        "gd0_weight" ~ 0, #everytime you see gd0_Weight, then replace with 0 
        "gd18_weight" ~ 18
    ))
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

## Pivot wider

Let’s make up an analysis result table.

``` r
analysis_df = 
  tibble(
    group = c("treatment", "treatment", "control", "control"),
    time  = c("pre", "post", "pre", "post"),
    mean = c(4, 10, 4.2, 5)
  )
```

Pivot wider for human readability

``` r
analysis_df |> 
  pivot_wider(
    names_from = time, #which column name are you using
    values_from = mean, #where are the values for that column coming from? the mean variable
  ) |> 
  knitr::kable() #almost always use this as a final step if someone will read the table
```

| group     | pre | post |
|:----------|----:|-----:|
| treatment | 4.0 |   10 |
| control   | 4.2 |    5 |

## Bind tables.

``` r
fellowship_ring = 
  read_excel("./data/LotR_Words.xlsx", range = "B3:D6") |>
  mutate(movie = "fellowship_ring")


two_towers = 
  read_excel("./data/LotR_Words.xlsx", range = "F3:H6") |>
  mutate(movie = "two_towers")


return_king = 
  read_excel("./data/LotR_Words.xlsx", range = "J3:L6") |>
  mutate(movie = "return_king")


lotr_df = 
  bind_rows(fellowship_ring, two_towers, return_king) |> 
  janitor::clean_names() |> 
  pivot_longer(
    cols = female:male,
    names_to = "sex",
    values_to = "words"
  ) |> 
  relocate(movie) |> #relocates column "movie" to the first column
  mutate(race = str_to_lower(race)) #changes race to lower case
```

## Join FAS datasets

Import `litters` dataset.

``` r
litters_df = 
  read_csv("data/FAS_litters.csv", na = c("NA", ".", "")) |> 
  janitor::clean_names() |> 
  mutate(
    wt_gain = gd18_weight - gd0_weight
  ) |> 
  separate(
    group , into = c("dose", "day_of_treatment"), sep = 3 #separate after 3 characters, can also ask it to look for a space or other special character
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

Import `pups` next!

``` r
pups_df = 
  read_csv("data/FAS_pups.csv", na = c("NA", ".", "")) |> 
  janitor::clean_names() |> 
  mutate(
    sex = case_match( #change the sex from 1 and 2 to male and female
      sex,
      1 ~ "male",
      2 ~ "female"
    )
  )
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Join the datasets!

``` r
fas_df = 
  left_join(pups_df, litters_df, by = "litter_number") |> 
  relocate(litter_number, dose, day_of_treatment) #reorder the columns starting with these as the first three in this order
```

Do final example on course website that includes the Learning
Assessment.

anti_join - is there anything that exists in one dataset that is not in
the other this is an important step to make sure that once datasets are
joined, you’re checking is everyone here.

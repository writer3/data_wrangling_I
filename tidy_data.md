Tidy Data
================

``` r
pulse_df = 
  haven::read_sas("data/public_pulse_data.sas7bdat") |> 
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

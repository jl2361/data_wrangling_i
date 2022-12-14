Data manipulation with `dplyr`
================

Once you’ve imported data, you’re going to need to do some cleaning up.

``` r
knitr::opts_chunk$set(
  collapse = TRUE,
  fig.width = 6,
  fig.asp = .6,
  out.width = "90%"
)
```

``` r
library(tidyverse)
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
## ✔ ggplot2 3.3.6      ✔ purrr   0.3.4 
## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
## ✔ tidyr   1.2.0      ✔ stringr 1.4.1 
## ✔ readr   2.1.2      ✔ forcats 0.5.2 
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()

options(tibble.print_min = 3)

litters_data = 
  read_csv("data/FAS_litters.csv")
## Rows: 49 Columns: 8
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (2): Group, Litter Number
## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

litters_data = 
janitor::clean_names(litters_data)

pups_data = read_csv("data/FAS_pups.csv")
## Rows: 313 Columns: 6
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (1): Litter Number
## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
janitor::clean_names(pups_data)
## # A tibble: 313 × 6
##   litter_number   sex pd_ears pd_eyes pd_pivot pd_walk
##   <chr>         <dbl>   <dbl>   <dbl>    <dbl>   <dbl>
## 1 #85               1       4      13        7      11
## 2 #85               1       4      13        7      12
## 3 #1/2/95/2         1       5      13        7       9
## # … with 310 more rows
```

### `select`

When you only need a subset of columns in a data tble; extracting only
what you need can helpfully de-clutter.

``` r
select(litters_data, group, litter_number, gd0_weight, pups_born_alive)
## # A tibble: 49 × 4
##   group litter_number gd0_weight pups_born_alive
##   <chr> <chr>              <dbl>           <dbl>
## 1 Con7  #85                 19.7               3
## 2 Con7  #1/2/95/2           27                 8
## 3 Con7  #5/5/3/83/3-3       26                 6
## # … with 46 more rows

select(litters_data, group:gd_of_birth)
## # A tibble: 49 × 5
##   group litter_number gd0_weight gd18_weight gd_of_birth
##   <chr> <chr>              <dbl>       <dbl>       <dbl>
## 1 Con7  #85                 19.7        34.7          20
## 2 Con7  #1/2/95/2           27          42            19
## 3 Con7  #5/5/3/83/3-3       26          41.4          19
## # … with 46 more rows
```

You can also specify columns you’d like to remove:

``` r
select(litters_data, -pups_survive)
## # A tibble: 49 × 7
##   group litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive pups_…¹
##   <chr> <chr>              <dbl>       <dbl>       <dbl>           <dbl>   <dbl>
## 1 Con7  #85                 19.7        34.7          20               3       4
## 2 Con7  #1/2/95/2           27          42            19               8       0
## 3 Con7  #5/5/3/83/3-3       26          41.4          19               6       0
## # … with 46 more rows, and abbreviated variable name ¹​pups_dead_birth
select(litters_data, -pups_survive, -group)
## # A tibble: 49 × 6
##   litter_number gd0_weight gd18_weight gd_of_birth pups_born_alive pups_dead_b…¹
##   <chr>              <dbl>       <dbl>       <dbl>           <dbl>         <dbl>
## 1 #85                 19.7        34.7          20               3             4
## 2 #1/2/95/2           27          42            19               8             0
## 3 #5/5/3/83/3-3       26          41.4          19               6             0
## # … with 46 more rows, and abbreviated variable name ¹​pups_dead_birth
```

You can rename variables as part of this process:

``` r
select(litters_data, GROUP=group, LiTtEr_NuMbEr = litter_number)
## # A tibble: 49 × 2
##   GROUP LiTtEr_NuMbEr
##   <chr> <chr>        
## 1 Con7  #85          
## 2 Con7  #1/2/95/2    
## 3 Con7  #5/5/3/83/3-3
## # … with 46 more rows
```

If all you want to do is rename something, you can use `rename` instead
of `select`. This will rename the variables you care about, and keep
everything else:

``` r
rename(litters_data, GROUP = group, LiTtEr_NuMbEr = litter_number)
## # A tibble: 49 × 8
##   GROUP LiTtEr_NuMbEr gd0_weight gd18_weight gd_of_birth pups_…¹ pups_…² pups_…³
##   <chr> <chr>              <dbl>       <dbl>       <dbl>   <dbl>   <dbl>   <dbl>
## 1 Con7  #85                 19.7        34.7          20       3       4       3
## 2 Con7  #1/2/95/2           27          42            19       8       0       7
## 3 Con7  #5/5/3/83/3-3       26          41.4          19       6       0       5
## # … with 46 more rows, and abbreviated variable names ¹​pups_born_alive,
## #   ²​pups_dead_birth, ³​pups_survive
```

There are some handy helper functions for `select`; read about all of
them using `?select_helpers`. I use `starts_with()`, `ends_with()`, and
`contains()` often, especially when there are variable names with
suffixes or other standard patterns:

``` r
select(litters_data, starts_with("gd"))
## # A tibble: 49 × 3
##   gd0_weight gd18_weight gd_of_birth
##        <dbl>       <dbl>       <dbl>
## 1       19.7        34.7          20
## 2       27          42            19
## 3       26          41.4          19
## # … with 46 more rows
select(litters_data, ends_with("weight"))
## # A tibble: 49 × 2
##   gd0_weight gd18_weight
##        <dbl>       <dbl>
## 1       19.7        34.7
## 2       27          42  
## 3       26          41.4
## # … with 46 more rows
```

I also frequently use `everything()`, which is handy for reorganizing
columns without discarding anything:

``` r
select(litters_data, litter_number, pups_survive, everything())
## # A tibble: 49 × 8
##   litter_number pups_survive group gd0_weight gd18_wei…¹ gd_of…² pups_…³ pups_…⁴
##   <chr>                <dbl> <chr>      <dbl>      <dbl>   <dbl>   <dbl>   <dbl>
## 1 #85                      3 Con7        19.7       34.7      20       3       4
## 2 #1/2/95/2                7 Con7        27         42        19       8       0
## 3 #5/5/3/83/3-3            5 Con7        26         41.4      19       6       0
## # … with 46 more rows, and abbreviated variable names ¹​gd18_weight,
## #   ²​gd_of_birth, ³​pups_born_alive, ⁴​pups_dead_birth
```

You will often filter using comparison operators (`>`, `>=`, `<`, `<=`,
`==`, and `!=`). You may also use `%in%` to detect if values appear in a
set (useful for characters), and `is.na()` to find missing values. The
results of comparisons are logical– the statement is `TRUE` or `FALSE`
depending on the values you compare – and can be combined with other
comparisons using the logical operators `&` and `|`, or negated using
`!`.

Some ways you might filter the litters data are:

-   `gd_of_birth == 20`
-   `pups_born_alive >= 2`
-   `pups_survive != 4`
-   `!(pups_survive == 4)`
-   `group %in% c("Con7", "Con8")`
-   `group == "Con7" & gd_of_birth == 20`

``` r
filter(litters_data, gd_of_birth == 20)
## # A tibble: 32 × 8
##   group litter_number gd0_weight gd18_weight gd_of_birth pups_…¹ pups_…² pups_…³
##   <chr> <chr>              <dbl>       <dbl>       <dbl>   <dbl>   <dbl>   <dbl>
## 1 Con7  #85                 19.7        34.7          20       3       4       3
## 2 Con7  #4/2/95/3-3         NA          NA            20       6       0       6
## 3 Con7  #2/2/95/3-2         NA          NA            20       6       0       4
## # … with 29 more rows, and abbreviated variable names ¹​pups_born_alive,
## #   ²​pups_dead_birth, ³​pups_survive

filter(litters_data, group == "Con7" & gd_of_birth == 20)
## # A tibble: 4 × 8
##   group litter_number   gd0_weight gd18_weight gd_of_b…¹ pups_…² pups_…³ pups_…⁴
##   <chr> <chr>                <dbl>       <dbl>     <dbl>   <dbl>   <dbl>   <dbl>
## 1 Con7  #85                   19.7        34.7        20       3       4       3
## 2 Con7  #4/2/95/3-3           NA          NA          20       6       0       6
## 3 Con7  #2/2/95/3-2           NA          NA          20       6       0       4
## 4 Con7  #1/5/3/83/3-3/2       NA          NA          20       9       0       9
## # … with abbreviated variable names ¹​gd_of_birth, ²​pups_born_alive,
## #   ³​pups_dead_birth, ⁴​pups_survive
```

A very common filtering step requires you to omit missing observations.
You *can* do this with `filter`, but I recommend using `drop_na` from
the `tidyr` package:

-   `drop_na(litters_data)` will remove any row with a missing value
-   `drop_na(litters_data, wt_increase)` will remove rows for which
    `wt_increase` is missing.

### `mutate`

The example below creates a new variable measuring the difference
between `gd18_weight` and `gd0_weight` and modifies the existing `group`
variable.

``` r
#litter_data2 = 
mutate(litters_data,
       wt_gain = gd18_weight - gd0_weight,
       group = str_to_lower(group),
       # wt_gain_kg = wt_gain * 2.2
       )
## # A tibble: 49 × 9
##   group litter_number gd0_weight gd18_…¹ gd_of…² pups_…³ pups_…⁴ pups_…⁵ wt_gain
##   <chr> <chr>              <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
## 1 con7  #85                 19.7    34.7      20       3       4       3    15  
## 2 con7  #1/2/95/2           27      42        19       8       0       7    15  
## 3 con7  #5/5/3/83/3-3       26      41.4      19       6       0       5    15.4
## # … with 46 more rows, and abbreviated variable names ¹​gd18_weight,
## #   ²​gd_of_birth, ³​pups_born_alive, ⁴​pups_dead_birth, ⁵​pups_survive
```

### `arrange`

In comparison to the preceding, arranging is pretty straightforward. You
can arrange the rows in your data according to the values in one or more
columns:

``` r
head(arrange(litters_data, group, pups_born_alive), 10)
## # A tibble: 10 × 8
##    group litter_number   gd0_weight gd18_weight gd_of_…¹ pups_…² pups_…³ pups_…⁴
##    <chr> <chr>                <dbl>       <dbl>    <dbl>   <dbl>   <dbl>   <dbl>
##  1 Con7  #85                   19.7        34.7       20       3       4       3
##  2 Con7  #5/4/2/95/2           28.5        44.1       19       5       1       4
##  3 Con7  #5/5/3/83/3-3         26          41.4       19       6       0       5
##  4 Con7  #4/2/95/3-3           NA          NA         20       6       0       6
##  5 Con7  #2/2/95/3-2           NA          NA         20       6       0       4
##  6 Con7  #1/2/95/2             27          42         19       8       0       7
##  7 Con7  #1/5/3/83/3-3/2       NA          NA         20       9       0       9
##  8 Con8  #2/2/95/2             NA          NA         19       5       0       4
##  9 Con8  #1/6/2/2/95-2         NA          NA         20       7       0       6
## 10 Con8  #3/6/2/2/95-3         NA          NA         20       7       0       7
## # … with abbreviated variable names ¹​gd_of_birth, ²​pups_born_alive,
## #   ³​pups_dead_birth, ⁴​pups_survive
```

You can also sort in descending order if you’d like.

``` r
head(arrange(litters_data, desc(group), pups_born_alive), 10)
## # A tibble: 10 × 8
##    group litter_number gd0_weight gd18_weight gd_of_bi…¹ pups_…² pups_…³ pups_…⁴
##    <chr> <chr>              <dbl>       <dbl>      <dbl>   <dbl>   <dbl>   <dbl>
##  1 Mod8  #7/82-3-2           26.9        43.2         20       7       0       7
##  2 Mod8  #97                 24.5        42.8         20       8       1       8
##  3 Mod8  #5/93/2             NA          NA           19       8       0       8
##  4 Mod8  #7/110/3-2          27.5        46           19       8       1       8
##  5 Mod8  #82/4               33.4        52.7         20       8       0       6
##  6 Mod8  #2/95/2             28.5        44.5         20       9       0       9
##  7 Mod8  #5/93               NA          41.1         20      11       0       9
##  8 Mod7  #3/82/3-2           28          45.9         20       5       0       5
##  9 Mod7  #5/3/83/5-2         22.6        37           19       5       0       5
## 10 Mod7  #106                21.7        37.8         20       5       0       2
## # … with abbreviated variable names ¹​gd_of_birth, ²​pups_born_alive,
## #   ³​pups_dead_birth, ⁴​pups_survive
```

### `%>%`

Piping! Control + Shift + M is the keyboard shortcut.

``` r
litters_data = 
  read_csv("data/FAS_litters.csv", col_types = "ccddiiii") %>%
  janitor::clean_names() %>%
  select(-pups_survive) %>%
  mutate(
    wt_gain = gd18_weight - gd0_weight,
    group = str_to_lower(group)) %>% 
  drop_na(wt_gain)

litters_data
## # A tibble: 31 × 8
##   group litter_number gd0_weight gd18_weight gd_of_birth pups_…¹ pups_…² wt_gain
##   <chr> <chr>              <dbl>       <dbl>       <int>   <int>   <int>   <dbl>
## 1 con7  #85                 19.7        34.7          20       3       4    15  
## 2 con7  #1/2/95/2           27          42            19       8       0    15  
## 3 con7  #5/5/3/83/3-3       26          41.4          19       6       0    15.4
## # … with 28 more rows, and abbreviated variable names ¹​pups_born_alive,
## #   ²​pups_dead_birth
```

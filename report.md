Times Higher Education College Ranking 2018
================

Load library
============

``` r
pacman::p_load(rvest, jsonlite, RCurl, data.table, ggplot2)
```

Get data
========

Use rvest to read the jquery script
-----------------------------------

``` r
webpage <- read_html(paste0(
  'https://www.timeshighereducation.com/rankings/united-states/', 
  '2018#!/page/0/length/25/sort_by/rank/sort_order/asc/cols/stats'))

t <- html_nodes(webpage, 'script') %>% 
  '['(9) %>% 
  html_text()
```

Use jsonlite to read the json object after inspecting the script
----------------------------------------------------------------

Found the json data at the following address

``` r
college_json <- fromJSON(paste0(
  'https://www.timeshighereducation.com/sites/default/files/the_data_rankings/', 
  'united_states_rankings_2018_limit0_efdb24148bae97278bbfe6ecfd71cdd9.json'))

college_dat <- college_json$data

#saveRDS(college_dat, './data/times_higher_education_2018_ranking.Rds')
```

Visualize the data
==================

``` r
setDT(college_dat)

dat <- copy(college_dat)

cols <- c('stats_fees_oos', 'stats_board', 'stats_salary')
dat[, (cols) := lapply(.SD, function(x) as.numeric(gsub('\\D', '', x))), .SDcols = cols]

dat <- dat[, .(rank_order, rank, name, scores_overall, scores_overall_rank,
               record_type, location, stats_fees_oos, stats_board, stats_salary)]

dat[, roi := stats_salary / stats_fees_oos]
dat[, rank_order := as.numeric(rank_order)]

ggplot(dat) +
  geom_point(aes(stats_fees_oos, stats_salary))
```

![](report_files/figure-markdown_github/unnamed-chunk-4-1.png)

``` r
ggplot(dat) + 
  geom_point(aes(stats_fees_oos, roi))
```

![](report_files/figure-markdown_github/unnamed-chunk-4-2.png)

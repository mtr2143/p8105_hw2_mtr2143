Homework 2
================
Matthew T. Russell
10/9/2021

# Problem 1

###### Reading and cleaning Mr. Trash Wheel sheet:

-   specify the sheet in the Excel file and to omit non-data entries
    (rows with notes/figures; columns containing notes) using arguments
    in `read_excel`
-   use reasonable variable names
-   omit rows that do not include dumpster-specific data
-   round the number of sports balls to the nearest integer

``` r
trash_data <- 
  read_excel("./Data/Trash-Wheel-Collection-Totals-8-6-19.xlsx", 
  sheet = "Mr. Trash Wheel") %>% 
  janitor::clean_names() %>% 
  select(-x15, -x16, -x17) %>% 
  filter(!(str_detect(month, "Total"))) %>% 
  mutate(sports_balls = round(sports_balls, 0))
```

###### Reading and cleaning 2018 and 2019 precipitation sheets:

-   omit rows without precipitation data
-   add a variable for year
-   combine precipitation datasets
-   convert month to a character variable

``` r
precip_2018_data <- 
  read_excel("./Data/Trash-Wheel-Collection-Totals-8-6-19.xlsx", 
  sheet = "2018 Precipitation", range = "A2:B14") %>% 
  janitor::clean_names() %>% 
  filter(!(is.na(total))) %>% 
  mutate(year = 2018)

precip_2019_data <- 
  read_excel("./Data/Trash-Wheel-Collection-Totals-8-6-19.xlsx", 
  sheet = "2019 Precipitation", range = "A2:B14") %>%
  janitor::clean_names() %>% 
  filter(!(is.na(total))) %>% 
  mutate(year = 2019)

precip_18_19_data <-
  bind_rows(precip_2018_data, precip_2019_data) %>% 
  mutate(month = month.name[month])
```

###### Summary of Mr. Trash Wheel and precipitation datasets:

There are 344 observations and 14 columns in the Mr. Trash Wheel dataset
and 18 observations and 3 columns in the combined precipitation dataset.
The Mr. Trash Wheel data tell us about the quantity of certain trash
items such as plastic bottles, cigarette butts, glass bottles, grocery
bags, chip bags, and sports balls. The median number of sports balls
consumed by Mr. Trash Wheel is 8. Meanwhile, the precipitation data tell
us about the total rainfall (inches) per month for the years 2018 and
2019. The total inches of rainfall in 2018 was 70.33.

# Problem 2

###### Reading and cleaning pols-months csv:

-   clean data
-   break up variable mon into integer variables year, month, and day
-   replace month number with month name
-   create a president variable taking values gop and dem
-   remove prez\_dem, prez\_gop, and day variables

``` r
pol_months_data <- read_csv(file = "./Data/pols-month.csv") %>% 
  separate(mon, into = c("year", "month", "day"), sep = "-", convert = T) %>% 
  mutate(month = month.name[month], 
         month = str_to_lower(month),
         president = prez_dem, 
         president = factor(president, levels = c(0,1), labels = c("gop", "dem"))) %>% 
  select(-c(prez_dem, prez_gop, day)) %>% 
  relocate(president, .after = month)
```

###### Reading and cleaning snp csv:

-   arrange columns so year and month are first

``` r
snp_data <- read_csv(file = "./Data/snp.csv") %>% 
  mutate(date = as.Date(date, "%m/%d/%y")) %>% 
  separate(date, into = c("year", "month", "day"), sep = "-", convert = T) %>% 
  mutate(month = month.name[month],
         month = str_to_lower(month),
         year = ifelse(year > 2021, year - 100, year)
        ) %>% 
  select(-day)
```

###### Tidying the unemployment csv:

-   change data from ‘wide’ to ‘long’ format

``` r
unemployment_data <- read_csv(file = "./Data/unemployment.csv") %>% 
  pivot_longer(
    Jan:Dec, 
    names_to = "month",
    values_to = "rate"
  ) %>% 
  mutate(
    month = match(month, month.abb), 
    month = month.name[month], 
    month = str_to_lower(month)
  ) %>% 
  janitor::clean_names()
```

###### Merge datasets:

``` r
pol_snp_unemp <- 
  left_join(pol_months_data, snp_data, by = c("year", "month")) %>% 
  left_join(unemployment_data, by = c("year", "month")) %>% 
  rename(unemp_rate = rate)
```

###### Summary of datasets:

The dataset from “pols-month” has provides information about the number
of republicans and democrats in congress as well the president’s
political party. The dataset from “snp” communicates the closing values
of the Standard & Poor’s stock market index. The dataset from
“unemployment” gives the percent unemployed by month and year. After
mergining “snp” and “unemployment” data into “pols-month” data, we have
822 observations with 9 columns about national political party
representation, the health of the stock market, and the unemployment
percentage from 1947 to 2015. Key variables include:

-   `president`: indicates whether president at that time was republican
    (gop) or democrat (dem)
-   `close`: closing value of Standard & Poor’s stock market index at
    that time
-   `unemp_rate`: percent unemployed at that time

# Problem 3

###### Reading and tidying baby names csv:

``` r
baby_names <- read_csv(file = "./Data/Popular_Baby_Names.csv") %>% 
  janitor::clean_names() %>% 
  mutate(
    across(gender:childs_first_name, str_to_lower), 
    ethnicity = ifelse(
      ethnicity %in% c("asian and pacific islander", "asian and paci"),
        "aapi", ethnicity
      ), 
    ethnicity = ifelse(
      ethnicity == "black non hisp", "black non hispanic", ethnicity
      ),
    ethnicity = ifelse(
      ethnicity == "white non hisp", "white non hispanic", ethnicity
      )
    ) %>% 
  distinct()
```

###### Creating table for popularity of ‘Olivia’ over time:

``` r
olivia_pop <- baby_names %>% 
  filter(childs_first_name == "olivia") %>% 
  select(-c(gender, childs_first_name, count)) %>%
  arrange(year_of_birth) %>% 
  pivot_wider(
    names_from = "year_of_birth", 
    values_from = "rank"
  )

library(knitr)
kable(olivia_pop, caption = "Popularity of 'Olivia' by ethnicity from 2011 to 2016")
```

| ethnicity          | 2011 | 2012 | 2013 | 2014 | 2015 | 2016 |
|:-------------------|-----:|-----:|-----:|-----:|-----:|-----:|
| aapi               |    4 |    3 |    3 |    1 |    1 |    1 |
| black non hispanic |   10 |    8 |    6 |    8 |    4 |    8 |
| hispanic           |   18 |   22 |   22 |   16 |   16 |   13 |
| white non hispanic |    2 |    4 |    1 |    1 |    1 |    1 |

Popularity of ‘Olivia’ by ethnicity from 2011 to 2016

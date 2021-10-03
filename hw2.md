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

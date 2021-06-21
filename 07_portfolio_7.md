# Working with databases

This exercise shows my work with SQL and PostgreSQL in DBeaver, and the relation from and to R in Rstudio.

Starting with some setup and loading and tidying of data in Rstudio.


```r
library(tidyverse) #version 1.3.1
library(dslabs) #version 0.7.4
library(RPostgres) #version 1.3.2
```


```r
df_flu <- read.csv("data/flu_data.csv", skip = 11)
df_dengue <- read.csv("data/dengue_data.csv", skip = 11)
df_gapminder <- gapminder
```


```r
df_flu <- df_flu %>% gather(key = "country", value = "activity", -Date)

df_dengue <- df_dengue %>% gather(key = "country", value = "activity", -Date)
```

Making sure the datasets coincide on variables, so they may be used together.


```r
df_flu <- separate(df_flu, Date, into = c("year", "month", "day"), sep = '-') 
df_flu$year <- as.integer(df_flu$year)
df_flu$country <- as.factor(df_flu$country)

df_dengue <- separate(df_dengue, Date, into = c("year", "month", "day"), sep = '-') 
df_dengue$year <- as.integer(df_dengue$year)
df_dengue$country <- as.factor(df_dengue$country)
```

And storing them for safe-keeping!


```r
write_csv(df_flu, here::here("data\\df_flu.csv"))
write_rds(df_flu, here::here("data\\df_flu.rds"))

write_csv(df_dengue, here::here("data\\df_dengue.csv"))
write_rds(df_dengue, here::here("data\\df_dengue.rds"))

write_csv(df_gapminder, here::here("data\\gapminder.csv"))
write_rds(df_gapminder, here::here("data\\df_gapminder.rds"))
```

Then inserting them into a premade PostgreSQL database in DBeaver.


```r
# con <- dbConnect(RPostgres::Postgres(),
#                dbname = "workflowsdb",
#                host = "localhost",
#                port = "5432",
#                user = "postgres",
#                password = "password")
# dbWriteTable(con, "df_flu", df_flu)
# dbWriteTable(con, "df_dengue", df_dengue)
# dbWriteTable(con, "df_gapminder", df_gapminder)
# dbDisconnect(con)
```

And inspecting the data to see if the transfer went properly.


```r
# in dbeaver
knitr::include_graphics(
  here::here(
    "images",
    "dbeaver_screenshot.png"
  )
)
```

<img src="C:/Users/ikben/OneDrive/Documenten/Data_science/workflows-portfolio/images/dbeaver_screenshot.png" width="418" />

```r
# in Rstudio
head(df_flu)
```

```
##   year month day   country activity
## 1 2002    12  29 Argentina       NA
## 2 2003    01  05 Argentina       NA
## 3 2003    01  12 Argentina       NA
## 4 2003    01  19 Argentina       NA
## 5 2003    01  26 Argentina       NA
## 6 2003    02  02 Argentina      136
```

```r
head(df_dengue)
```

```
##   year month day   country activity
## 1 2002    12  29 Argentina       NA
## 2 2003    01  05 Argentina       NA
## 3 2003    01  12 Argentina       NA
## 4 2003    01  19 Argentina       NA
## 5 2003    01  26 Argentina       NA
## 6 2003    02  02 Argentina       NA
```

```r
head(df_gapminder)
```

```
##               country year infant_mortality life_expectancy fertility
## 1             Albania 1960           115.40           62.87      6.19
## 2             Algeria 1960           148.20           47.50      7.65
## 3              Angola 1960           208.00           35.98      7.32
## 4 Antigua and Barbuda 1960               NA           62.97      4.43
## 5           Argentina 1960            59.87           65.39      3.11
## 6             Armenia 1960               NA           66.86      4.55
##   population          gdp continent          region
## 1    1636054           NA    Europe Southern Europe
## 2   11124892  13828152297    Africa Northern Africa
## 3    5270844           NA    Africa   Middle Africa
## 4      54681           NA  Americas       Caribbean
## 5   20619075 108322326649  Americas   South America
## 6    1867396           NA      Asia    Western Asia
```

The gapminder dataset needed some cleaning and ofcourse this new, clean dataset needed to be put in the database as well.


```r
clean_gapminder <- df_gapminder %>% na.omit()

# con <- dbConnect(RPostgres::Postgres(),
#                dbname = "workflowsdb",
#                host = "localhost",
#                port = "5432",
#                user = "postgres",
#                password = "password")
# dbWriteTable(con, "clean_gapminder", clean_gapminder)
# dbDisconnect(con)

knitr::include_graphics(
  here::here(
    "images",
    "db_screenshot2.png"
  )
)
```

<img src="C:/Users/ikben/OneDrive/Documenten/Data_science/workflows-portfolio/images/db_screenshot2.png" width="412" />

Then joining them using SQL and putting the joined dataset back in Rstudio to show some desciptive statistics and graphs.


```r
knitr::include_graphics(here::here(
  "images",
  "db_screenshot3.png"
))
```

<img src="C:/Users/ikben/OneDrive/Documenten/Data_science/workflows-portfolio/images/db_screenshot3.png" width="533" />

```r
joined_dfs <- read.csv("data/joined_dfs_202106171226.csv")

summary(joined_dfs)
```

```
##  infant_mortality  flu_activity    dengue_activity       year     
##  Min.   :12.70    Min.   :  56.0   Min.   :0.0040   Min.   :2002  
##  1st Qu.:14.80    1st Qu.: 178.0   1st Qu.:0.0270   1st Qu.:2005  
##  Median :16.60    Median : 268.0   Median :0.0560   Median :2007  
##  Mean   :22.65    Mean   : 441.6   Mean   :0.0982   Mean   :2007  
##  3rd Qu.:27.05    3rd Qu.: 539.0   3rd Qu.:0.1080   3rd Qu.:2009  
##  Max.   :53.70    Max.   :3041.0   Max.   :1.0000   Max.   :2011  
##                   NA's   :2551     NA's   :366                    
##    country         
##  Length:97768      
##  Class :character  
##  Mode  :character  
##                    
##                    
##                    
## 
```

```r
tidy_dfs <- joined_dfs %>% gather('infant_mortality', 'flu_activity', 'dengue_activity', key = "group", value = "activity", na.rm = TRUE)

tidy_dfs %>% filter(activity <= 540) %>% ggplot(aes(x = group, y = activity)) +
  geom_boxplot(aes(fill = group)) +
  labs(title = "Boxplot of the activity per group",
       x = "",
       y = "Activity",
       caption = "Data from the flu_data, dengue_data and gapminder datasets") +
  theme_minimal()
```

<img src="07_portfolio_7_files/figure-html/stats_graphs-2.png" width="672" />

```r
tidy_dfs %>% ggplot(aes(x = country, y = activity)) +
  geom_boxplot(aes(fill = country)) +
  labs(title = "Boxplot of the activity per country",
       x = "",
       y = "Activity",
       caption = "Data from the flu_data, dengue_data and gapminder datasets") +
  theme_minimal()
```

<img src="07_portfolio_7_files/figure-html/stats_graphs-3.png" width="672" />

```r
tidy_dfs %>% filter(group == "flu_activity") %>% ggplot(aes(x = year, y = activity)) +
  geom_smooth() +
  labs(title = "Graph of the average flu activity per year",
       x = "Year",
       y = "Activity",
       caption = "Data from the flu_data dataset") +
  theme_minimal()
```

```
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
```

<img src="07_portfolio_7_files/figure-html/stats_graphs-4.png" width="672" />

STAT545-HW04
================
Shirlett
October 6, 2017

Reshaping
---------

### Activity \#4 - One row per year for each life expectancy min and max rank

``` r
# This is taken from the HW windows function example giving the country with both the
# lowest and highest life expectancy (in Asia)
gaptble <- gapminder %>%
  filter(continent == "Asia") %>%
  select(year, country, lifeExp) %>%
  group_by(year) %>%
  filter(min_rank(desc(lifeExp)) < 2 | min_rank(lifeExp) < 2) %>% 
  arrange(year)

knitr::kable(gaptble, format.args = list(decimal.mark = '.'),  digits=2, caption="Countries in Asia with Minimum and Maximum Life Expectancy \nby Year - Long Format")
```

|  year| country     |  lifeExp|
|-----:|:------------|--------:|
|  1952| Afghanistan |    28.80|
|  1952| Israel      |    65.39|
|  1957| Afghanistan |    30.33|
|  1957| Israel      |    67.84|
|  1962| Afghanistan |    32.00|
|  1962| Israel      |    69.39|
|  1967| Afghanistan |    34.02|
|  1967| Japan       |    71.43|
|  1972| Afghanistan |    36.09|
|  1972| Japan       |    73.42|
|  1977| Cambodia    |    31.22|
|  1977| Japan       |    75.38|
|  1982| Afghanistan |    39.85|
|  1982| Japan       |    77.11|
|  1987| Afghanistan |    40.82|
|  1987| Japan       |    78.67|
|  1992| Afghanistan |    41.67|
|  1992| Japan       |    79.36|
|  1997| Afghanistan |    41.76|
|  1997| Japan       |    80.69|
|  2002| Afghanistan |    42.13|
|  2002| Japan       |    82.00|
|  2007| Afghanistan |    43.83|
|  2007| Japan       |    82.60|

``` r
#Get only the countries with the minimum life expectancies and add new columns
firsttble <- gaptble %>%
  do(head(., n = 1)) %>%
  mutate(LifeType='Minimum_Life', LifeCountry='Minimum_Life_Country') 
  
#Get only the countries with the maximum life expectancies and add new columns
secondtble <- gaptble %>%
  do(tail(., n = 1)) %>%
  mutate(LifeType='Maximum_Life', LifeCountry='Maximum_Life_Country')

#Combine the rows again from the first and second table by binding/appending
recombo_tble <- bind_rows(firsttble, secondtble) %>%
            select(year, LifeCountry, country, LifeType, lifeExp)

#Spread the country value using LifeCountry as key
firstspread <- recombo_tble %>%
  spread(LifeCountry, country) 
  
 
#Do second Spread with the LifeExp value using LifeType as key 
final_spread <- firstspread %>%
  spread(LifeType,lifeExp,  fill=0, drop=TRUE) %>%
  group_by(year) %>%
  summarize(Min_Life_Country=last(Minimum_Life_Country), Min_Life_Expectancy=last(Minimum_Life), Max_Life_Country=first(Maximum_Life_Country),  Max_Life_Expectancy=first(Maximum_Life))  


knitr::kable(final_spread, digits=2, caption="Countries in Asia with Minimum and Maximum Life Expectancy \nby Year - Wide Format")
```

|  year| Min\_Life\_Country |  Min\_Life\_Expectancy| Max\_Life\_Country |  Max\_Life\_Expectancy|
|-----:|:-------------------|----------------------:|:-------------------|----------------------:|
|  1952| Afghanistan        |                  28.80| Israel             |                  65.39|
|  1957| Afghanistan        |                  30.33| Israel             |                  67.84|
|  1962| Afghanistan        |                  32.00| Israel             |                  69.39|
|  1967| Afghanistan        |                  34.02| Japan              |                  71.43|
|  1972| Afghanistan        |                  36.09| Japan              |                  73.42|
|  1977| Cambodia           |                  31.22| Japan              |                  75.38|
|  1982| Afghanistan        |                  39.85| Japan              |                  77.11|
|  1987| Afghanistan        |                  40.82| Japan              |                  78.67|
|  1992| Afghanistan        |                  41.67| Japan              |                  79.36|
|  1997| Afghanistan        |                  41.76| Japan              |                  80.69|
|  2002| Afghanistan        |                  42.13| Japan              |                  82.00|
|  2007| Afghanistan        |                  43.83| Japan              |                  82.60|

JOINING
-------

### Activity \#1 - Explore different types of join between gapminder and new dataframe

``` r
#Create a new dataframe called gapcont

continent <- c('Africa','Americas','Antarctica','Asia', 'Europe', 'Oceania')
size_sq_km <- c(30244049, 42068068, 14000000, 44391162, 10354636, 7686884)
hemisphere <- c('Eastern','Western', 'Western','Eastern', 'Eastern', 'Eastern')

gapcont <- data.frame(continent, size_sq_km, hemisphere)

knitr::kable(gapcont, format.args = list(decimal.mark = '.', big.mark = ","), digits=2, caption="GapCont Table showing Area and Hemisphere for all Continents")
```

| continent  |  size\_sq\_km| hemisphere |
|:-----------|-------------:|:-----------|
| Africa     |    30,244,049| Eastern    |
| Americas   |    42,068,068| Western    |
| Antarctica |    14,000,000| Western    |
| Asia       |    44,391,162| Eastern    |
| Europe     |    10,354,636| Eastern    |
| Oceania    |     7,686,884| Eastern    |

``` r
# Left Join
# This results in every single available row in gapminder being returned since all 
# continents in gapminder have a match in gapcont. It also includes the variables
# that match from the second dataframe.
l_join <- left_join(gapminder, gapcont, by='continent')
l_join %>%
  filter(year==2007 | is.na(year)) %>%
  select(continent, pop, size_sq_km, hemisphere) %>%
  arrange(continent) %>%
  group_by(continent, size_sq_km, hemisphere) %>%
  summarise(total_pop=sum(as.numeric(pop))) %>%
  knitr::kable(format.args = list(decimal.mark = '.', big.mark = ","),  digits=2, caption="Left Join Result Summary")
```

| continent |  size\_sq\_km| hemisphere |     total\_pop|
|:----------|-------------:|:-----------|--------------:|
| Africa    |    30,244,049| Eastern    |    929,539,692|
| Americas  |    42,068,068| Western    |    898,871,184|
| Asia      |    44,391,162| Eastern    |  3,811,953,827|
| Europe    |    10,354,636| Eastern    |    586,098,529|
| Oceania   |     7,686,884| Eastern    |     24,549,947|

``` r
# Right Join
# This results in every single available row in gapminder in addition to the one row 
# in gapcont that does not have a match which is Antarctica.
r_join <- right_join(gapminder, gapcont, by='continent') 

r_join %>%
  filter(year==2007 | is.na(year)) %>%
  select(continent, pop, size_sq_km, hemisphere) %>%
  arrange(continent) %>%
  group_by(continent, size_sq_km, hemisphere) %>%
  summarise(total_pop=sum(as.numeric(pop))) %>%
  knitr::kable(format.args = list(decimal.mark = '.', big.mark = ","),  digits=2, caption="Right Join Result Summary")
```

| continent  |  size\_sq\_km| hemisphere |     total\_pop|
|:-----------|-------------:|:-----------|--------------:|
| Africa     |    30,244,049| Eastern    |    929,539,692|
| Americas   |    42,068,068| Western    |    898,871,184|
| Antarctica |    14,000,000| Western    |             NA|
| Asia       |    44,391,162| Eastern    |  3,811,953,827|
| Europe     |    10,354,636| Eastern    |    586,098,529|
| Oceania    |     7,686,884| Eastern    |     24,549,947|

``` r
# Inner Join
# This results in every single available row in gapminder being returned since all 
# continents match in both dataframes. All variables from both tables are combined
# in the order of the inner join function.
i_join <- inner_join(gapminder, gapcont, by='continent')
i_join %>%
  filter(year==2007 | is.na(year)) %>%
  select(continent, pop, size_sq_km, hemisphere) %>%
  arrange(continent) %>%
  group_by(continent, size_sq_km, hemisphere) %>%
  summarise(total_pop=sum(as.numeric(pop))) %>%
  knitr::kable(format.args = list(decimal.mark = '.', big.mark = ","),  digits=2, caption="Inner Join Result Summary")
```

| continent |  size\_sq\_km| hemisphere |     total\_pop|
|:----------|-------------:|:-----------|--------------:|
| Africa    |    30,244,049| Eastern    |    929,539,692|
| Americas  |    42,068,068| Western    |    898,871,184|
| Asia      |    44,391,162| Eastern    |  3,811,953,827|
| Europe    |    10,354,636| Eastern    |    586,098,529|
| Oceania   |     7,686,884| Eastern    |     24,549,947|

``` r
# Full Join
# This results in all rows in both the gapminder and the gapcont being returned
f_join <- full_join(gapminder, gapcont, by='continent')
f_join %>%
  filter(year==2007 | is.na(year)) %>%
  select(continent, pop, size_sq_km, hemisphere) %>%
  arrange(continent) %>%
  group_by(continent, size_sq_km, hemisphere) %>%
  summarise(total_pop=sum(as.numeric(pop))) %>%
  knitr::kable(format.args = list(decimal.mark = '.', big.mark = ","),  digits=2, caption="Full Join Result Summary")
```

| continent  |  size\_sq\_km| hemisphere |     total\_pop|
|:-----------|-------------:|:-----------|--------------:|
| Africa     |    30,244,049| Eastern    |    929,539,692|
| Americas   |    42,068,068| Western    |    898,871,184|
| Antarctica |    14,000,000| Western    |             NA|
| Asia       |    44,391,162| Eastern    |  3,811,953,827|
| Europe     |    10,354,636| Eastern    |    586,098,529|
| Oceania    |     7,686,884| Eastern    |     24,549,947|

``` r
# Semi-Join
# This results in only the rows in which continent matches in both the gapminder and 
# gapcont but no variables from gapcont are returned
s_join <- semi_join(gapminder, gapcont, by='continent')
s_join %>%
  filter(year==2007 | is.na(year)) %>%
  select(continent, pop) %>%
  arrange(continent) %>%
  group_by(continent) %>%
  summarise(total_pop=sum(as.numeric(pop))) %>%
  knitr::kable(format.args = list(decimal.mark = '.', big.mark = ","),  digits=2, caption="Semi Join Result Summary")
```

| continent |     total\_pop|
|:----------|--------------:|
| Africa    |    929,539,692|
| Americas  |    898,871,184|
| Asia      |  3,811,953,827|
| Europe    |    586,098,529|
| Oceania   |     24,549,947|

``` r
# Anti-Join with gapminder as the first dataset
# This results in no rows being returned since all of the continents in gapminder have # a match in gapcont
a_join <- anti_join(gapminder, gapcont, by='continent')
a_join %>%
  knitr::kable(format.args = list(decimal.mark = '.', big.mark = ","),  digits=2, caption="Anti Join Result Summary - Gapminder as First Dataframe")
```

Table: Anti Join Result Summary - Gapminder as First Dataframe

country continent year lifeExp pop gdpPercap -------- ---------- ----- -------- ---- ----------

``` r
# Anti-join with gapminder as the second dataset
# This results in only one row being returned since one of the continents in gapcont  
# do not have a match in gapminder
a_join_sec <- anti_join(gapcont, gapminder,  by='continent')

knitr::kable(a_join_sec, format.args = list(decimal.mark = '.', big.mark = ","),  digits=2, caption="Anti Join Result Summary - Gapminder as Second Dataframe")
```

| continent  |  size\_sq\_km| hemisphere |
|:-----------|-------------:|:-----------|
| Antarctica |       1.4e+07| Western    |

### Activity 3 - Merging and Matching

``` r
# Merging

# This merges two data frames horizontally based on at least one similar variable. The # result of doing a full merge of the gapminder dataset and the second gapcont 
# dataframe is the same as full join in dyplr where all values and all rows are 
# retained. There #are also parameters that can be added to the merge function to  
# mimic a left join, #right join or inner join using "all*".

#Same results as a full join
all_mrge <- merge(gapminder, gapcont, "continent", all=TRUE)
str(all_mrge)
```

    ## 'data.frame':    1705 obs. of  8 variables:
    ##  $ continent : Factor w/ 6 levels "Africa","Americas",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ country   : Factor w/ 142 levels "Afghanistan",..: 20 17 20 52 36 46 36 86 36 17 ...
    ##  $ year      : int  1992 1962 1997 1962 1977 1967 1962 2002 1972 1957 ...
    ##  $ lifeExp   : num  54.3 37.8 52.2 35.8 46.5 ...
    ##  $ pop       : int  12467171 4919632 14195809 3140003 228694 489004 89898 31167783 178848 4713416 ...
    ##  $ gdpPercap : num  1793 723 1694 686 3082 ...
    ##  $ size_sq_km: num  30244049 30244049 30244049 30244049 30244049 ...
    ##  $ hemisphere: Factor w/ 2 levels "Eastern","Western": 1 1 1 1 1 1 1 1 1 1 ...

``` r
#Same results as a inner join
inner_mrge <- merge(gapminder, gapcont, "continent", all=FALSE)
str(inner_mrge)
```

    ## 'data.frame':    1704 obs. of  8 variables:
    ##  $ continent : Factor w/ 5 levels "Africa","Americas",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ country   : Factor w/ 142 levels "Afghanistan",..: 20 17 20 52 36 46 36 86 36 17 ...
    ##  $ year      : int  1992 1962 1997 1962 1977 1967 1962 2002 1972 1957 ...
    ##  $ lifeExp   : num  54.3 37.8 52.2 35.8 46.5 ...
    ##  $ pop       : int  12467171 4919632 14195809 3140003 228694 489004 89898 31167783 178848 4713416 ...
    ##  $ gdpPercap : num  1793 723 1694 686 3082 ...
    ##  $ size_sq_km: num  30244049 30244049 30244049 30244049 30244049 ...
    ##  $ hemisphere: Factor w/ 2 levels "Eastern","Western": 1 1 1 1 1 1 1 1 1 1 ...

``` r
#Same results as a left join
left_mrge <- merge(gapminder, gapcont, "continent", all.x=TRUE)
str(left_mrge)
```

    ## 'data.frame':    1704 obs. of  8 variables:
    ##  $ continent : Factor w/ 5 levels "Africa","Americas",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ country   : Factor w/ 142 levels "Afghanistan",..: 20 17 20 52 36 46 36 86 36 17 ...
    ##  $ year      : int  1992 1962 1997 1962 1977 1967 1962 2002 1972 1957 ...
    ##  $ lifeExp   : num  54.3 37.8 52.2 35.8 46.5 ...
    ##  $ pop       : int  12467171 4919632 14195809 3140003 228694 489004 89898 31167783 178848 4713416 ...
    ##  $ gdpPercap : num  1793 723 1694 686 3082 ...
    ##  $ size_sq_km: num  30244049 30244049 30244049 30244049 30244049 ...
    ##  $ hemisphere: Factor w/ 2 levels "Eastern","Western": 1 1 1 1 1 1 1 1 1 1 ...

``` r
#Same results as a right join
right_mrge <- merge(gapminder, gapcont, "continent", all.y=TRUE)
str(right_mrge)
```

    ## 'data.frame':    1705 obs. of  8 variables:
    ##  $ continent : Factor w/ 6 levels "Africa","Americas",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ country   : Factor w/ 142 levels "Afghanistan",..: 20 17 20 52 36 46 36 86 36 17 ...
    ##  $ year      : int  1992 1962 1997 1962 1977 1967 1962 2002 1972 1957 ...
    ##  $ lifeExp   : num  54.3 37.8 52.2 35.8 46.5 ...
    ##  $ pop       : int  12467171 4919632 14195809 3140003 228694 489004 89898 31167783 178848 4713416 ...
    ##  $ gdpPercap : num  1793 723 1694 686 3082 ...
    ##  $ size_sq_km: num  30244049 30244049 30244049 30244049 30244049 ...
    ##  $ hemisphere: Factor w/ 2 levels "Eastern","Western": 1 1 1 1 1 1 1 1 1 1 ...

``` r
#Match

# This function returns a vector of the position of the first occurrence of a 
# continent from gapcont in gapminder. The example below shows that Africa is first 
# found on row 25 in the gapminder dataset. This function does not join together a 
# dataset, but it is useful for selecting the appropriate join/merge function, 
# especially left or right join. The match function will also inform the user of any 
# missing variables that should have been present in the two datasets when results 
# show NA.

match(gapcont$continent, gapminder$continent)
```

    ## [1] 25 49 NA  1 13 61
R Notebook
================

## Loading Data

``` r
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.1.0     ✔ purrr   0.2.5
    ## ✔ tibble  1.4.2     ✔ dplyr   0.7.8
    ## ✔ tidyr   0.8.2     ✔ stringr 1.3.1
    ## ✔ readr   1.2.1     ✔ forcats 0.3.0

    ## ── Conflicts ───────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(foreign)
library(naniar)

mc78 = "/Users/grayden/Documents/Grad_School/Capstone/Mental Health/Medical Conditions/2007-2008.XPT"
mc910 = "/Users/grayden/Documents/Grad_School/Capstone/Mental Health/Medical Conditions/2009-2010.XPT"


dem78 = "/Users/grayden/Documents/Grad_School/Capstone/Mental Health/Demographics/2007-2008.XPT"
dem910 = "/Users/grayden/Documents/Grad_School/Capstone/Mental Health/Demographics/2009-2010.XPT"

dep78 = "/Users/grayden/Documents/Grad_School/Capstone/Mental Health/Depression Survey/07-08.XPT"
dep910 = "/Users/grayden/Documents/Grad_School/Capstone/Mental Health/Depression Survey/09-10.XPT"

crp78 <- "/Users/grayden/Documents/Grad_School/Capstone/Mental Health/CRP/07-08.XPT"
crp910 <- "/Users/grayden/Documents/Grad_School/Capstone/Mental Health/CRP/09-10.XPT"

mc78 = read.xport(mc78)
mc910 = read.xport(mc910)
dem78 = read.xport(dem78)
dem910 = read.xport(dem910)
dep78 <- read.xport(dep78)
dep910 <- read.xport(dep910)
crp78 <- read.xport(crp78)
crp910 <- read.xport(crp910)
```

## Cleaning Data

``` r
#Joining Demographic Datasets
dem = dem78 %>% rbind(dem910)


#(Cleaning, then) Joining Medical Condition Datasets
mc78 = mc78[,c(1, 16:18)]
mc910 = mc910[,c(1, 17:19)]
mc78 = mc78 %>% replace_with_na_at(.vars=c('MCQ190','MCQ160A'), condition=~.x >=7) %>% drop_na(MCQ160A)
mc910 = mc910 %>% replace_with_na_at(.vars=c('MCQ191','MCQ160A'), condition=~.x >=7) %>% drop_na(MCQ160A)
mc78$MCQ190 = ifelse(mc78$MCQ190 %in% NA, NA, ifelse(mc78$MCQ190==1, '0', ifelse(mc78$MCQ190==2,'1','2')))
mc910$MCQ191 = ifelse(mc910$MCQ191 %in% NA, NA, ifelse(mc910$MCQ191==1, '0', ifelse(mc910$MCQ191==2,'1','2')))
mc78$MCQ190 = factor(mc78$MCQ190, levels=c('0','1','2'), labels=c('Rheumatoid', 'Osteo','Other'))
mc910$MCQ191 = factor(mc910$MCQ191, levels=c('0','1','2'), labels=c('Rheumatoid','Osteo', 'Other'))
mc78 = mc78[!(mc78$MCQ160A==1 & mc78$MCQ190 %in% NA),]
mc910 = mc910[!(mc910$MCQ160A==1 & mc910$MCQ191 %in% NA),]
names(mc78)[4] = "MCQ190"
names(mc910)[4] = "MCQ190"
mc = mc78 %>% rbind(mc910)

#Joining Depression Datasets
dep78[,c(2:11)] = dep78[,c(2:11)] %>% replace_with_na_all(condition=~.x >=7)
dep910[,c(2:11)] = dep910[,c(2:11)] %>% replace_with_na_all(condition=~.x >=7)

dep = dep78 %>% rbind(dep910)


#Joininig CRP Datasets
crp = crp78 %>% rbind(crp910)
```

``` r
#Joining datasets by responder id

data <- dem %>% inner_join(mc) %>% inner_join(dep) %>% inner_join(crp)
```

    ## Joining, by = "SEQN"
    ## Joining, by = "SEQN"
    ## Joining, by = "SEQN"

``` r
#Adjusting Sample Weights for Multi-Year Period
data$WTINT2YR <- data$WTINT2YR/2
data$WTMEC2YR <- data$WTMEC2YR/2
```

``` r
#Exporting data
write.csv(data, 'MC<>DEPR<>DEM<>CRP.csv')
```

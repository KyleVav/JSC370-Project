JSC370 Midterm
================
2024-03-05

## Introduction

  
In recent years there has been an increasing pessimism about the
prospect of owning a home, with younger generations dissatisfied how
older generations own homes well they cannot afford to. The goal of this
project is to examine differences in home ownership between generations
through home owernship rates by year, age, and locations. The proposed
research question is: Have home ownership rates in the United States
changed between the generations of Baby Boomers, Gen X, Millenials, and
Gen Z?

  
The dataset that was used to answer this question is a subset of the
IPSUM CPS dataset \[citation\]. The dataset comes from the United States
Census Current Population SUrvey (CPS) and its supplemental Annual
Social and Economic Supplement survey (ASEC). The CPS is preformed
monthly to different households in the US while the ASEC supplement is
preformed annualy in March. The dataset contains both household and
individual level information, meaning some variables refer to entire
household information such as whether a house is owned or rented, while
other variables refer to individual information such as a persons age.

  
As the entire IPUMS CPS dataset is large and contains many variables
that are irrelevant to this analysis, a subset of the data was used.
Only data from the years 1976 to 2023 was used. The variables used in
the dataset were:

  
-YEAR which is the year an observation was from

-AGE which is the age of an individual,

-OWNERSHIP which is whether the household that an individual belongs to
is rented/owned,

-RELATE which is the relation to their relation to the householder of
the household an individual belongs to

-STATEFIP which is the FIP code of the state were the observation was
from

-COUNTY which is the FIP code of the county were the observation was
from

## Methods

  
IPUMS CPS allows users to create their own subsets, called data
extracts, of the full CPS dataset that contain variables and year ranges
of interest. To create a data extract, a user must select variables and
year ranges and then wait for a data extract to be generated; once the
extract is generated it can be downloaded. The creation and retrieval of
the data extract used in this analysis was preformed through the IPUMS
API. To create the extract through the API, a post request was made
using the httr R package with a formatted JSON string that specified the
year range and variables of interest. Once the data extract request was
submitted, a get request was used to check whether the data extract had
been generated. Finally, when the data extract was complete, the data
files were downloaded from the URLs provided from the content of the get
request. The dataset was then loaded in R using the ipumsr package
provided by IPUMS for loading IPUMS data into R.

  
In addition to the key variables mentioned in the introduction, IPUMS
provides weights for calculating individual and household level
statistics to mitigate the sampling bias of the the survey, these
weights were also included in the dataset as the variables ASECWTH for
household level statistics and ASECWT for individual level statistics.

  
Once the dataset was loaded into R, it’s dimensions were checked for how
many observations were present in the dataset and whether the number of
variables was consistent with what was requested in the data extract.
Checking the dimension showed that the dataset contains 8,293,750
observations with each observation being an individual who belongs to a
household. The head, tail, and variable types were also checked to make
sure the dataset was imported correctly.

  
Once the dataset was confirmed to have been imported correctly, the
variables were checked for missing values and suspicious values. All
values for AGE, YEAR, RELATE, OWNERSHP, and STATEFIP had no missing
values and were all in their expected range; however, the variable
COUNTY had 3,159,092 missing values and 2,991,865 values of 0 which is
not a recognized county FIP code. Checking the IPUMS data dictionary
confirmed that a value of 0 was used for counties that did not meet a
population threshold due to concerns about deanonymizing data. Further
investigation also showed that all the missing values in the COUNTY
variable occurred before 1995.

  
The number of samples in ages \<19 and \>90 were small which creates
outliers in the home ownership rates within these age ranges. The number
of people buying homes in these age ranges is also very small so
comparisons within these age ranges are irrelevant to the research
question. Additionally, In the United States, people cannot own property
until they reach the age of majority which is 19 in some states.
Therefore, these age ranges were removed from the data set. After
removing all observations with age less than 19 and greater than 90
there were 5,796,909 observations left.

  
For the analysis, a new variable had to be created referring the
generation that individual belongs to. Next, a variable to indicate
which generation an observation belongs to was created; the generation
age ranges were chosen to be consistent with an article by Pew Research
Center, the age ranges were: 1946-1964 (Baby Boomer), 1965-1980 (Gen X),
1981-1996 (Millennial), 1997-2012 (Gen Z) \[2\]. The generation variable
was created by subtracting the AGE variable from the YEAR variable to
get a birth year and then checking which generation range it was in.
Finally, the frequencies of each generation was checked to see if the
variable was create properly, as you might expect, Gen Z had the least
amount of observations at 86,029 while Baby Boomers had the most
2,147,536 due to the data set containing many years before any
individual in Gen Z was born.

  
Once the data was cleaned, the new variable for Generation was created,
the analysis could be preformed. To answer the research question, home
ownership rates had to be calculated across generations. For this
analysis, a home ownership rate was defined:

$$\text{Home Ownership Rate (%)} = 100 \cdot \frac{\text{Owner Occupied Households}}{\text{Total Occupied Households}}$$

  
This definition is consistent the definition given by the Census Housing
Vacancy Survey \[2\]. As the dataset provides weights for calculating
household level statistics, the home ownership rate was calculated as
the the sum of the weights associated with an owner occupied household
divided by the sum of weights associated with an occupied household.
Finally, a generations home ownership rate is calculated the same,
except only households owned/occupied by an individual in that
generation are counted.

  
To find owner occupied households within the dataset both the OWNERSHP
and RELATE variable had to be used. The OWNERSHP variable in the dataset
refers to whether the household the observation belongs to is owned or
rented; this does not mean that the individual is a homeowners as, for
example, an individual living with parents who own a home would have a
value ownership rather than rented. Instead, both the variables OWNERSHP
and RELATE have to be used to find homeowners. If OWNERSHP indicates
that the observation belong to a household that owns their home and
RELATE indicates that the observation is the householder then that
observation is considered a homeowner.

  
Finally, visualizations and summaries were created to explore the
differences in homeownership rates between generations. First, table
summaries were created to show homeownership rates by generation in
different years. As you might expect, younger generations would have
lower home ownership rates simply due to being younger rather than any
differences between the generation, therefore looking at the trend of
homeownership by year and age will give a better idea of the difference
between generations. To explore the trend of homeownership rate over
time, a line chart was created to show homeownership by year with a
trend line for each generation. Similarly, to explore the trend of
homeownership by age, a line chart was created to show homeownership by
age with a trend line for each generation. To explore difference in
homeownership in locations by generation, a map visualization was
created to show home ownership at the state level, and at the county
level. To create these map visualizations, geojson files of USA state
and county maps that included FIPS data were used. Home ownership rates
were calculated by state/county and then this data was merged into the
geojson files using the FIPS codes. For the county map, some counties
were not identified, these counties were merged together within state
borders to form a map that contained homeownerhip rates in all
identified counties, and home ownership rates in states not including
identified counties within the state. Similarly, any county with \<50
observations in any generation was combined with the non-identified
counties within state borders.

## Preliminary Results

| YEAR | Baby Boomer |    Gen X |    Gen Z | Millenial |
|-----:|------------:|---------:|---------:|----------:|
| 2018 |    76.94919 | 66.50245 | 21.25693 |  40.61062 |
| 2019 |    77.22946 | 66.92789 | 22.49573 |  43.23064 |
| 2020 |    78.74693 | 69.01683 | 22.56327 |  47.94009 |
| 2021 |    78.47346 | 69.02732 | 23.36040 |  48.67208 |
| 2022 |    77.82291 | 69.72024 | 25.71410 |  51.45640 |
| 2023 |    77.99646 | 71.36857 | 25.87277 |  54.23662 |

  
This table shows the homeownership rates by generation in recent years.
Since the older generations like Baby Boomers and Millennials have been
alive longer, and likely have more wealth, we would expect to see that
their home ownership rate is much higher as shown in the table. We also
see that the home ownership rate for Millennials has increased
significantly in this time period, around a 14% difference with a
signicant increase in 2020. Most Millennials would be in their late 20s
and early 30s which is around the age of the average first time
homebuyer in the United States. There were also historically low
interest rates between 2019-2021 which could have contributed to this
increase. Finally, we can also see that Baby Boomers were the only
generation to have a decrease in homeownership rates within this time
period, this could be due to aging populations selling their home to
move in with family or into care homes. In the future, we would expect
to see the Baby Boomer home ownership rate continue to fall. Overall, we
see that Millennials and Gen Z have the largest increase in home
ownership rates within this time period.

| AGE | Baby Boomer |    Gen X |    Gen Z | Millenial |
|----:|------------:|---------:|---------:|----------:|
|  25 |    31.25654 | 26.84121 | 28.95171 |  27.38984 |
|  30 |    50.61442 | 48.34606 |       NA |  42.27259 |
|  35 |    60.50193 | 58.64897 |       NA |  55.59943 |
|  40 |    67.73174 | 63.52479 |       NA |  61.27793 |

  
This table shows the homeownership rates at certain ages by generation.
As the oldest members of Gen Z were only 26 in 2023, their is no data
available in the later ages. We can see that Baby Boomers have the
highest home ownership rate at all ages while Millennials have the
lowest across all ages except 25. While Gen Z only has data at the age
25, it has the second highest home ownership rate at this age. There
does appear to be a trend of homeownership rates dropping at all ages as
the generations get younger; however, Gen Z could disrupt this trend if
their rates continue to be high. Without further data on Gen Z, we
cannot know if this trend will continue or Gen Z will outperform
Millennials and/or Gen X.

![](JSC370-midterm_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->  
These plots show overall trends of home ownership by year and age. We
can see that home ownership rates tend to stay fairly similar over time.
This makes sense as outside of large economic shifts we would expect to
have the proportion of people owning homes stay fairly constant as
younger people buy houses and older people move out.We can also see that
home ownership rates by age increases over time and then starts to
flatten out around 50, before starting to decrease at around 80. Again,
this would make sense as people would want to buy homes once they have a
stable income and career which would likely occur in the age range
25-40. Then since most people who had the means to buy a house or wanted
to buy a house likely already did by 50, the rate starts to flatten.
Finally, once a person is older, they may start to sell their homes and
move in with family or care home. Interestingly, we also see an outlier
at around 19 where the rate starts off higher. The slightly elevated
rate at 19 could be due to inheritance where an individual is not
legally allowed to own a property until they reach age of majority.

![](JSC370-midterm_files/figure-gfm/unnamed-chunk-20-1.png)<!-- -->  
This plot shows the trend of home ownership by year between each
generation. We can see that Baby Boomer rate is has flattened
significantly since around 2005 and the Gen X was starting to flatten
around 2005 as well although it has started to increase again since
around 2015; the sub prime mortgage crisis occurred between 2007 and
2010 and could have contributed to this decrease in the rate of change
of the Gen X home ownership rate. For Millennials, we can see that the
rate has been increasing and is starts to increase faster around 2015.
Finally, for Gen Z there is not a lot of data, but we can see that it is
starting to increase and in 2020 it had the rate started to increase
faster.

![](JSC370-midterm_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

  
This plot shows the trend of home ownership by age between each
generation. The plot provides a good comparison of home ownership rates
by generation. We can see that Baby Boomers have consistently
outperformed all other generations past the age of 26 and we can also
see that Millenials have under preformed all other generations past the
age of 26. The Gen X rate tracks fairly closely to the Baby Boomer rate
before it starts to under preform in the 30s age range and onward. Gen Z
out preforms both Millennial and Gen X for the age range where data is
available, while the Gen Z line flattens towards the age of 26, without
further data we do not know whether Gen Z will outperform or under
preform other generations. Low intrest rates from 2019-2021, when older
members of Gen Z would have started thinking about buying a home, could
have contributed to the high home ownership rates of Gen Z relative to
the other generations.

![](JSC370-midterm_files/figure-gfm/unnamed-chunk-23-1.png)<!-- -->

  
This map shows home ownership rates by state and generation. We can see
that Baby Boomers have fairly high home ownership rates across all
states. The states with the lowest home ownership rates for Baby Boomers
are California, New York, and Hawaii. For Gen X, we see lower home
ownership than Baby Boomers. We also see that Midwestern states and
non-coastal states tend to have higher home ownership rates, which could
be due to affordability. For Millennials, we can see again that home
ownership is lower than Gen X, but we also see that the states with the
lowest home ownership rates are similar to the states with the lowest
home ownership rates for Gen Z. Finally, for Gen Z the states with the
lowest home ownership rates are fairly similar to Millenials and Gen X,
but surprisingly Florida is one of the states with the highest home
ownership rate which is unique to Gen Z.

![](JSC370-midterm_files/figure-gfm/unnamed-chunk-28-1.png)<!-- -->

  
Similar to the previous map, this map shows home ownership rates by
geography and generation. This map includes counties that were
identified in the dataset, many counties were not identified in the
dataset as their population was below a threshold were there are
concerns about deanonymizing the data. All counties that were not
identified in the dataset and any county with less than 50 observations
in any generation are combined into the state and labeled “State w/o
Counties”. Similar patterns can be seen to the previous state map;
however, the difference between urban and rural home ownership rates are
more obvious in this map, with rural areas tending to have higher home
ownership rates across all generations.

## Summary

  
The analysis compared home ownership rates across generations by
investigating trends in home ownership rate over time, age, and
geography.

  
The trend of home ownership rate over years between generations was
similar, with fast growth followed by a flattening of the curve;
however, some differences between home ownership rate over year trend
could be seen, such as the increase in Gen X home ownership rate slowing
down between the years 2005 and 2015, potentially due to housing market
conditions in this period. While some differences were present, the
overall trends appeared similar, showing that the trend home ownership
rate over time has not significantly changed between generations.

  
The trend of home ownership rate over age between generations was also
similar. Although some deviations could be seen in the trends between
generations. Millennials tended to have the lowest home ownership rates
at all ages when compared to the other generations, while Baby Boomers
tended to have the highest rates at all ages. Gen X tracked closely with
Baby Boomer between the ages 19-35 but started to under preform at later
ages. Surprisingly, Gen Z was had the highest home ownership rates in
ages 19-23 and the second highest in ages 23-26 with Baby Boomers as the
highest in that age range. The trends between each generation being
similar and the high home ownership rates of Gen Z indicate that home
ownership has not increased or declined significantly between
generations despite Baby Boomers and Gen X tending to have higher rates.

  
The analysis of home ownership rates by state and by county showed that
home ownership rates tended to be similar across geography between the
generations. Home ownership rates in less populous states and counties,
such as Midwestern states or non-identified counties, tended to be
higher across all generations; although there were some interesting
differences, such as Florida being one of the states with the highest
home ownership rate for Gen Z compared to all other generations were
Florida was not one of the states with high home ownership rate.

  
Overall, the analysis showed that home ownership rates by year, age, and
geography have remained fairly similar between generations. Although the
home ownership rates do not appear to have increased or declined
significantly, this does not necessarily mean the housing market
remained the same. This analysis does not take into account any income,
mortgage rates, or pricing data, so it cannot give an accurate picture
of the housing market or housing affordability. Further analysis that
include more economic data would provide more cohesive information about
housing market between generations.

## Citations:

  
IPUMS: Sarah Flood, Miriam King, Renae Rodgers, Steven Ruggles, J.
Robert Warren, Daniel Backman, Annie Chen, Grace Cooper, Stephanie
Richards, Megan Schouweiler, and Michael Westberry. IPUMS CPS: Version
11.0 \[dataset\]. Minneapolis, MN: IPUMS, 2023.
<https://doi.org/10.18128/D030.V11.0>

  
Generation Ranges:

\[1\]
<https://www.pewresearch.org/short-reads/2019/01/17/where-millennials-end-and-generation-z-begins/>

  
Home Ownership Definition:

\[2\] <https://www.census.gov/housing/hvs/data/histtabs.html>

  
Geojson for states:

<https://rstudio.github.io/leaflet/json/us-states.geojson>

  
Geojson for counties:

<https://gist.github.com/sdwfrost/d1c73f91dd9d175998ed166eb216994a>

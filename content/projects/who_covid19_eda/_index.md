---
title: "Exploratory Data Analysis: WHO COVID-19 data"
author: Sam Larkin
date: 2022-12-21
---


## Introduction

This report outlines the process followed during an Exploratory Data Analysis
(EDA) on data published by the World Health Organisation (WHO) data on daily
COVID-19 cases and deaths by date reported to the WHO. [^who_data]

The dataset (as retrieved on 2022-12-12) contains 254064 records (this dataset continues
to be updated and will contain more records if retrieved at a later date), each
with 8 attributes:

| Name                | Type    | Description                                             |
| :---                | :---    | :----------                                             |
| `Date_reported`     | Date    | Date of reporting to WHO                                |
| `Country_code`      | String  | ISO Alpha-2 country code                                |
| `Country`           | String  | Country, territory, area                                |
| `WHO_region`        | String  | WHO region offices                                      |
| `New_cases`         | Integer | New confirmed cases                                     |
| `Cumulative_cases`  | Integer | Cumulative confirmed cases reported to the WHO to date  |
| `New_deaths`        | Integer | New confirmed deaths                                    |
| `Cumulative_deaths` | Integer | Cumulative confirmed deaths reported to the WHO to date |


## Missing Data

### Null values

The dataset has no null values in any column, with the exception of the
`Country_code` column, which is missing 1072 values (0.42% of rows in the
dataset).

Upon inspecting the rows with missing data, it seems that only records with
`Country` values of "Other" are missing. The WHO state that this category
indicates:

> Other: cases and deaths reported from international conveyances, included in
> global totals but not reflected in epidemiological curves as not associated
> with a country or region.

This data has been retained, in order to avoid incorrectly adjusting the global
figures on COVID-19 cases and deaths.

That being said, the data with no ISO Alpha-2 country code [^country_codes] has
not been mapped in the chloropleth maps shown below, as these country codes are
the means by which the COVID-19 data from the WHO was merged with the geographical
representation of each country's boundary, and with its estimated population
(both of which are available in a single GeoJSON file from the World Bank).
[^wb_countries]


### Data not recorded

Although -- as described above -- there is very little missing data in the dataset
in terms of null values, there is likely to be a significant amount of data
which is completely missing from the dataset (having never been recorded and
reported to the WHO).

Cases of COVID-19 which have gone untested and undiagnosed -- and deaths
resulting from cases of COVID-19 which were not detected -- will not have been
reported to the WHO.

There are many possible causes for failure to identify COVID-19 cases and to
report them to the WHO. These may include, but are not limited to:

* shortages of testing kits and facilities
* lack of access to medical care of any kind
* lack of access to communications networks (e.g., the internet) and therefore
  to avenues for reporting positive test results
* deliberate avoidance of testing in order to avoid local, national and international
  movement restrictions, or due to a financial imperative to stay in work
  whilst infected
* failure to self-test due to a lack of concern about the implications of
  having COVID-19 (especially following widespread vaccinations and easing of
  restrictions)
* failure of national agencies to report cases to the WHO for political reasons

The data presented below should be viewed with this limitation in mind.


## Data Cleaning

The data supplied by the WHO was already very clean and complete (with the caveat
regarding unreported data as described above).

The values in the `Date_reported` column were parsed to python datetime objects
(from string representations of dates as provided in the original dataset) using pandas'
`to_datetime` method in order to facilitate easy manipulation of the date data.


## Data Exploration

### Investigation of COVID-19 cases and deaths by WHO region

The WHO divides the world into six regions, and has a regional office for each
of these regions. These are:

| WHO region code   | Description                                   |
| :---------------- | :----------                                   |
| AFRO              | Regional Office for Africa                    |
| AMRO              | Regional Office for the Americas              |
| SEARO             | Regional Office for South-East Asia           |
| EURO              | Regional Office for Europe                    |
| EMRO              | Regional Office for the Eastern Mediterranean |
| WPRO              | Regional Office for the Western Pacific       |

Figure 1 shows the total number of (all-time) COVID-19 cases reported to the
WHO by region. The data indicates that Europe and the Americas have recorded
the largest numbers of cases of COVID-19 since the beginning of the global
pandemic, whilst the Eastern Mediterranean and Africa have recorded the fewest.

{{< figure src="./images/covid_cases_by_region.png" align=center caption="COVID-19 cases by WHO region" >}}

Figure 2 shows the total number of (all-time) COVID-19 deaths reported to the
WHO by region. The number of deaths in each region correlates as expected with
the number of cases in each region -- larger numbers of cases predict
larger numbers of deaths.

Note that the order of the regions is not exactly the same as that shown in
Figure 1. For example, whilst Europe recorded higher case numbers
than the Americas, the Americas recorded a larger number of deaths. This may
reflect differences in the overall standard of healthcare provision between
regions (for example, that healthcare provision in Europe is better in general
than healthcare in the Americas -- or at least that treatment of COVID-19 cases
was more effective in Europe).

{{< figure src="./images/covid_deaths_by_region.png" align=center caption="COVID-19 deaths by WHO region" >}}

Figure 3 shows the number of COVID-19 deaths per 100 cases reported to the WHO
by region. The result shown is interesting in that, whilst (as shown in Figures
1 and 2) the WHO's Africa region reported the lowest numbers of cases and
deaths, it is the region with the highest death rate for confirmed COVID-19
cases.

This may indicate several things. One interpretation of this result is that
access to testing and healthcare facilities in the Africa region was lower than
in other regions, meaning that both more confirmed cases of COVID-19 resulted
in death, but also that many more mild cases of COVID-19 went untested
and undiagnosed and therefore were not reported to the WHO (so are not present in
the dataset), thus disproportionately weighting the Africa region's data with
more serious -- and fatal -- cases and artificially inflating the number of
deaths per 100 cases relative to the other regions.

{{< figure src="./images/covid_deaths_per_100_cases_by_region.png" align=center caption="COVID-19 deaths per 100 cases by region" >}}

Figure 4 shows a scatter plot of total (all-time) COVID-19 deaths against
total (all-time) COVID-19 cases by country. The best-fit line indicates the
positive correlation between numbers of cases and numbers of deaths noted above
(see Figures 1 and 2) more clearly. Note that the international data noted by a
country value of "Other" has been excluded.

{{< figure src="./images/corr_cases_deaths_countries.png" align=center caption="Correlation between numbers of cases and numbers of deaths" >}}

### Global COVID-19 cases and deaths over time

Figure 5 shows the cumulative totals of COVID-19 cases and deaths reported to
the WHO over the course of the pandemic. Whilst numbers of
cases are in the order of hundreds of millions, deaths are in the order of
millions. At a glance, this tells us that the global death rate for confirmed
cases of COVID-19 is around 1% (1.29% to be precise).

As described above, it is
likely that the true death rate is lower than this (as many mild cases of
COVID-19 will not have been confirmed and reported to the WHO).

We can also see a significant increase in the rate of increase of global
cumulative case numbers in January 2022 (with a less pronounced increase in the
rate of increase of deaths lagging behind by about a month), which had been
increasing nearly linearly throughout 2021.

{{< figure src="./images/covid_global_cumulative.png" align=center caption="Cumulative global COVID-19 figures" >}}

Figure 6 shows the numbers of new confirmed COVID-19 cases and deaths reported
to the WHO each month over the course of the pandemic. Data for December 2022
is partial and has therefore been excluded to avoid confusion.

The spike in global case numbers in January 2022 described above can be seen
more clearly in this visualisation. We can also see the positive impact of the
global vaccination program in the lower numbers of deaths per month in late
2021 and 2022 (despite similar or higher numbers of cases per month).

{{< figure src="./images/covid_global_monthly.png" align=center caption="Monthly global COVID-19 figures" >}}


### UK COVID-19 cases and deaths over time

Figure 7 shows the cumulative totals of COVID-19 cases and deaths in the UK
reported to the WHO over the course of the pandemic. This data at a
national level is much less smooth than the global data (which is likely smoothed by
the local spikes in cases and deaths happening at slightly different times).

{{< figure src="./images/covid_uk_cumulative.png" align=center caption="Cumulative UK COVID-19 figures" >}}

Figure 8 shows the numbers of new confirmed COVID-19 cases and deaths in the UK reported
to the WHO each month over the course of the pandemic. Data for December 2022
is partial and has therefore been excluded to avoid confusion.

The impact of the vaccination program is even more pronounced here -- with
spikes in case numbers corresponding to significant spikes in deaths in 2020
and early 2021, and far fewer deaths resulting from the larger spike in cases
in winter 2021-2022.

A visualisation like this one could be compared with the dates of the national
lockdowns[^uk_lockdowns] to better understand the reasoning behind those lockdowns, and to
evaluate their effectiveness.

{{< figure src="./images/covid_uk_monthly.png" align=center caption="Monthly UK COVID-19 figures" >}}


### Investigation of COVID-19 cases and deaths per 100,000 population by country

Figure 9 shows the number of COVID-19 cases per 100,000 population for each
country in the WHO dataset.

As noted in the Missing Data section, the figures not associated with a
particular country values have been excluded from the maps below.

Western European countries immediately stand out as having recorded very high numbers of
cases relative to their populations.

{{< figure src="./images/covid_cases_per_100000_map.png" align=center caption="COVID-19 cases per 100,000 population chloropleth map" >}}

Figure 10 shows the number of COVID-19 deaths per 100,000 population for each
country in the WHO dataset.

Peru immediately jumps out as having been particularly badly affected in terms
of COVID-19 deaths relative to its population. This result was discussed by the
BBC in June 2021. [^bbc_peru]

{{< figure src="./images/covid_deaths_per_100000_map.png" align=center caption="COVID-19 deaths per 100,000 population chloropleth map" >}}




[^who_data]: WHO,
             [WHO Coronavirus (COVID-19) Dashboard](https://covid19.who.int/data),
             accessed 2022-12-12

[^country_codes]: GOV.UK,
                  [ISO Country Codes](https://www.gov.uk/government/publications/iso-country-codes--2),
                  accessed 2022-12-15

[^wb_countries]: World Bank,
                 [World Bank Official Boundaries](https://datacatalog.worldbank.org/search/dataset/0038272/World-Bank-Official-Boundaries),
                 accessed 2022-12-13
                 
[^uk_lockdowns]: Institute for Government,
                 [Timeline of UK government coronavirus lockdowns and restrictions](https://www.instituteforgovernment.org.uk/charts/uk-government-coronavirus-lockdowns),
                 accessed 2022-12-21
                 
[^bbc_peru]: BBC,
             [Covid: Why has Peru been so badly hit?](https://www.bbc.co.uk/news/world-latin-america-53150808),
             accessed 2022-12-21

---
title: Exploratory Data Analysis on the UCI Automobile Dataset
author: Sam Larkin
date: 2022-12-08T23:19:26Z
draft: false
---


## Introduction

This report outlines the process followed during an Exploratory Data Analysis
(EDA) of the automobile dataset, as supplied in
[`automobile.txt`](./automobile.txt) in the same directory as this report. The
dataset is also available online at
[`http://archive.ics.uci.edu/ml/datasets/Automobile`](http://archive.ics.uci.edu/ml/datasets/Automobile).

The dataset contains data relating to 26 different attributes of 205
automobiles. The attributes are:

| Name                | Description                                           | Type        |        Units |
|:--------------------|:------------------------------------------------------|:------------|-------------:|
| `symboling`         | insurance risk rating                                 | Categorical |          n/a |
| `make`              | name of manufacturer                                  | Categorical |          n/a |
| `fuel-type`         | type of fuel used                                     | Categorical |          n/a |
| `aspiration`        | aspiration type                                       | Categorical |          n/a |
| `num-of-doors`      | number of doors in words e.g. `"four"`                | Categorical |          n/a |
| `body-style`        | style of car                                          | Categorical |          n/a |
| `drive-wheels`      | drive wheels                                          | Categorical |          n/a |
| `engine-location`   | position of engine                                    | Categorical |          n/a |
| `fuel-system`       | fuel delivery system e.g. multi-point fuel injection  | Categorical |          n/a |
| `engine-type`       | type of engine e.g. dual overhead camshaft (`"dohc"`) | Categorical |          n/a |
| `num-of-cylinders`  | number of cylinders in engine in words e.g. `"six"`   | Categorical |          n/a |
| `normalized-losses` | average loss payment per insured vehicle per year     | Numerical   |          USD |
| `wheel-base`        | distance between centres of front and rear wheels     | Numerical   |       inches |
| `length`            | total length of vehicle                               | Numerical   |       inches |
| `width`             | width of vehicle                                      | Numerical   |       inches |
| `height`            | height of vehicle                                     | Numerical   |       inches |
| `curb-weight`       | kerb/curb weight i.e. weight including fluids         | Numerical   |       pounds |
| `engine-size`       | engine capacity (total swept volume)                  | Numerical   | cubic inches |
| `bore`              | internal diameter of engine cylinders                 | Numerical   |       inches |
| `stroke`            | swept length of cylinder (stroke)                     | Numerical   |       inches |
| `compression-ratio` | ratio of min cylinder volume to max cylinder volume   | Numerical   |          n/a |
| `horsepower`        | maximum engine power output                           | Numerical   |   horsepower |
| `peak-rpm`          | maximum crankshaft rotational speed                   | Numerical   |          rpm |
| `city-mpg`          | average fuel efficiency for city driving              | Numerical   |          mpg |
| `highway-mpg`       | average fuel efficiency for highway driving           | Numerical   |          mpg |
| `price`             | price of vehicle                                      | Numerical   |          USD |


## Missing Data

The original dataset contains a number of missing values (represented by the
character `?`). The locations of the missing values in the dataset can be seen in
the visualisation in Figure 1.

{{< figure src="./images/automobile_nulls.png" align=center caption="Missing data visualisation">}}

The accompanying table shows the number of null values in each column as an absolute
number and as a percentage of the number of rows in the dataset.

| Column              | Null values | Percentage null |              Approach |
|:--------------------|:-----------:|:---------------:|----------------------:|
| `normalized-losses` |    `41`     |     `20.0`      |         drop (column) |
| `num-of-doors`      |     `2`     |     `0.98`      | impute or drop (rows) |
| `bore`              |     `4`     |     `1.95`      |           drop (rows) |
| `stroke`            |     `4`     |     `1.95`      |           drop (rows) |
| `horsepower`        |     `2`     |     `0.98`      |           drop (rows) |
| `peak-rpm`          |     `2`     |     `0.98`      |           drop (rows) |
| `price`             |     `4`     |     `1.95`      |           drop (rows) |

As so much of the data in the `normalized-losses` column is missing, imputation
would be difficult and is probably inappropriate. The best course of action is
to drop the column (rather than to lose 20% of the rows in the dataset).

It seems likely that the number of doors for the two vehicles which are missing
this data can be imputed from some other characteristics in the dataset,
notably the `body-style`. All the convertibles and hardtops in the dataset have
two doors, whilst all the wagons have four. Hatchbacks are more likely to have
2 doors and sedans are more likely to have 4 doors. That being said, imputing
data like this can skew the outcomes of an EDA so the rows with missing data
have been dropped from the dataset.

The missing values in the remaining columns would be much more difficult (and
inaccurate) to impute so the rows with null values in these columns should be
dropped from the dataset.

After dropping this missing data, we are left with a dataset containing
information on 25 different attributes of 193 automobiles, with no missing values.


## Data Cleaning
 
The first step in cleaning the data, after handling the missing data as
described above, was to remove any duplicated rows, to avoid these skewing the
results of the EDA. No duplicate rows were identified in the automobiles
dataset.

Upon inspecting the data types of the remaining data, some of the types seemed
like an unusual choice for representing the underlying data.

The `num-of-doors` and `num-of-cylinders` columns clearly represent integer
values (a car may have 2 or 4 doors etc.). These columns were changed from
object (string) values to integer values to better represent these numerical
attributes.

The `horsepower`, `peak-rpm` and `price` columns are all integer values in the
raw data, but were converted to floats when reading the data with pandas. This
is due to the fact that [`NaN` is a float in numpy and
pandas](https://pandas.pydata.org/pandas-docs/stable/user_guide/integer_na.html).
This data was cast back to integers to better represent the underlying
raw data.

The name of the car manufacturer "Alfa Romeo" was misspelled as `alfa-romero`
in the original dataset. This spelling error was corrected for clarity.


## Data Stories and Visualisations

### Symboling data -- relative risk to insure each vehicle

The distribution of the `symboling` data in the dataset as seen in Figure
2 is skewed towards positive `symboling` numbers. As this
`symboling` data represents the relative risk to associated with insuring each
vehicle, this result indicates that insurance companies are more likely to rate
vehicles listed in this dataset as high risk, rather than low risk. It is
possible that risky vehicles are overrepresented in the dataset or that
insurance companies are simply cautious.

{{< figure src="./images/symboling.png" align=center caption="Distribution of `symboling` (relative insurance risk) data">}}

The box plot in Figure 3 indicates that there is a
clear relationship between the `body-style` of a vehicle and its `symboling`
value. Convertibles and hardtops are rated as being very risky to insure by
insurance companies, likely because they are often driven faster and more
recklessly.

Hatchbacks are also rated as relatively risky, with only wagons having a median
symboling value of less than 0 (which represents average insurance risk),
likely because they are usually family vehicles, used to transport children, 
and are therefore driven more carefully.

{{< figure src="./images/symboling_body_style.png" align=center caption="Relationship between `symboling` and `body-style`">}}

 
### Make

The bar plot in Figure 4 shows the number of vehicles of each `make`
(manufacturer) in the dataset. From this, we can see that Japanese export
vehicles are the most common in the dataset (Toyota, Nissan, Mitsubishi,
Subaru and Mazda i.e. the 5 most common makes in the dataset are all from
Japan). European export vehicles are the next most common.

{{< figure src="./images/make.png" align=center caption="Vehicle `make` sorted by frequency in the dataset">}}

From the visualisation in Figure 5, we can see that whilst
Japanese vehicles are the most common in the dataset, European exports are the
most expensive. These brands are viewed as luxury and charge a premium for
their vehicles.

Interestingly American made vehicles are at the bottom of the
list in terms of price in the dataset. This is likely because the dataset is
from an American source and the cost of import inflates the prices of the
non-American vehicles relative to the American ones.

{{< figure src="./images/price_by_make.png" align=center caption="Mean vehicle `price` by `make` sorted by price">}}

In order to investigate this relationship further, we could do some (very
simple) feature engineering to include the country of origin of each vehicle in
the dataset.


### Fuel type

From the pie chart, in Figure 6, we can see that the vast
majority of vehicles in the dataset use petrol (gasoline), rather than diesel
as their fuel.

{{< figure src="./images/fuel_type.png" align=center caption="Proportion of vehicles in the dataset by `fuel-type`">}}

Interestingly, even though there are relatively few diesel vehicles in the data
set, we can see from the visualisation in Figure 7 
that there is a marked differece between the typical `compression-ratio` for a
petrol engine and a diesel engine.

The mean `compression-ratio` for petrol engine vehicles in the dataset is
around 8.85, but for diesel vehicles, the mean `compression-ratio` is
approximately 21.97. This is nearly 2.5 times the mean value for the petrol
vehicles in the dataset.

This difference is due to the different volatilities of the two `fuel-types`.
Petrol is much more volatile, which means that a lower `compression-ratio` is
needed to achieve ignition of the fuel-air mixture in the cylinder. In fact,
an excessive `compression-ratio` can cause 'knocking' (autoignition of the
fuel-air mixture at the wrong time in the engine cycle) in petrol engines.

{{< figure src="./images/fuel_type_compression.png" align=center caption="Distribution of `compression-ratio` by `fuel-type`">}}

Looking at Figure 8, we can draw several conclusions. Firstly,
that there is a strong positive correlation between `engine-size` and
`horsepower`, which is intuitive, but also that diesel powered vehicles tend to
need a larger engine to achieve the same power output relative to petrol
vehicles. This is much less obvious.

{{< figure src="./images/hp_es_ft.png" align=center caption="Pair plot showing the relationships between `engine-size` and `horsepower` by `fuel-type`">}}


### Price

It has already been shown that the make of a vehicle and its country of origin
have an impact on its price. Some other features which seems to have
relationship with price are identified below.

Figure 9 shows the mean price for vehicles of each `body-style`
category. A vehicle's `body-style` seems to have some impact on its price. Note
that this chart does not reflect the hierarchy of `symboling` (insurance risk)
by `body-style` as described above, which suggests (as previously noted) that a
vehicle's price is not the only deciding factor in determining insurance risk.

{{< figure src="./images/price_body_style.png" align=center caption="Bar plot showing mean `price` by `body-style` category">}}

Figure 10 shows the mean price of vehicles in the dataset grouped
by their drive-wheels category. We can see that front wheel drive vehicles are
the least expensive, as they have the least complex drivetrain. The data shows
that almost all the vehicles (190 of 193) have their engine at the front of the
vehicle, so the power is easier to transfer from the engine to the wheels with
a less complex drivetrain in front wheel drive vehicles.

{{< figure src="./images/price_drive_wheels.png" align=center caption="Bar plot showing mean `price` by `drive-wheels` category">}}

Figure 11 shows that the different `engine-type` categories have different
average prices. This is due to the relative complexity (and therefore) cost of
each type of engine to build.

{{< figure src="./images/price_engine_type.png" align=center caption="Bar plot showing mean `price` by `engine-type` category">}}

The pair plot in Figure 12, showing the relationships between a vehicle's `price`,
`horsepower` and `highway-mpg` shows clearly that there is a strong positive
correlation between a vehicle's `horsepower` and its `price`. There is a
strongly negative correlation between `highway-mpg` (the vehicle's
fuel-efficiency on the highway) and its `price`. More powerful vehicles tend to
have larger, more complex engines, and are sometimes turbocharged, all of which
contributes to their increased `price` (and to decreased fuel-efficiency on the
highway).

{{< figure src="./images/price_hmpg_hp.png" align=center caption="Pair plot showing relationships between `price`, `horsepower`, and `highway-mpg` values">}}

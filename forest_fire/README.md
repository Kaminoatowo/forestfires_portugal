# Explore forest fire with R

Use the data set `forestfires.csv`, which can be downloaded [here](https://archive.ics.uci.edu/ml/machine-learning-databases/forest-fires/)

Install `tidyverse`: 
```install.packages("tidyverse")```
Load `tidyverse`: 
```library(tidyverse)```

Import it to R: In RStudio, click on the Workspace tab, and then on “Import Dataset” -> “From text file”.

## The data

The data we'll be working with in this guided project is associated with a scientific research paper on predicting the occurrence of forest fires in Portugal using modeling techniques.

In this project we will focus on data visualization. Exploring data visually is often the first step data scientists take when working with new data.

Here are descriptions of the variables in the data set and the range of values for each taken from the paper:

-   **X**: X-axis spatial coordinate within the Montesinho park map: 1 to 9
-   **Y**: Y-axis spatial coordinate within the Montesinho park map: 2 to 9
-   **month**: Month of the year: 'jan' to 'dec'
-   **day**: Day of the week: 'mon' to 'sun'
-   **FFMC**: Fine Fuel Moisture Code index from the FWI system: 18.7 to 96.20
-   **DMC**: Duff Moisture Code index from the FWI system: 1.1 to 291.3
-   **DC**: Drought Code index from the FWI system: 7.9 to 860.6
-   **ISI**: Initial Spread Index from the FWI system: 0.0 to 56.10
-   **temp**: Temperature in Celsius degrees: 2.2 to 33.30
-   **RH**: Relative humidity in percentage: 15.0 to 100
-   **wind**: Wind speed in km/h: 0.40 to 9.40
-   **rain**: Outside rain in mm/m2 : 0.0 to 6.4
-   **area**: The burned area of the forest (in ha): 0.00 to 1090.84

The acronym FWI stands for "fire weather index", a method used by scientists to quantify risk factors for forest fires.

Each row of the table represents a fire event. It is described spatially (with the `X` and `Y` coordinates), temporally (with `month` and `day`), thermodynamically (using the temperature (`temp`), the relative humidity (`RH`), wind speed (`wind`), outside rain (`rain`) and the burned area (`area`)). More specific features are the Fine Fuel Moisture Code (`FFMC`) - a rate of the moisture content of litter and other cured fine fuels-, the Duff Moisture Code (`DMC`) - a numeric rating of the average moisture content of loosely compacted organic layers of moderate depth -, the Drought Code (`DC`) - drying deep into the soil -, and the Initial Spread Index (`ISI`) - a measure of the head fire indicator and rate of fire spread.

Order the `month` and `day` features:

```{r}
> month_order <- c("jan", "feb", "mar", "apr", 
                    "may", "jun", "jul", "aug", 
                    "sep", "oct", "nov", "dec")
> days_order <- c("mon", "tue", 
                    "wed", "thu", 
                    "fri", "sat", 
                    "sun")
> forestfires <- forestfires %>% mutate(
    month = factor(month, levels = month_order), 
    day = factor(day, levels = days_order)
    )
>
```

### When?

Group the data by the `month` and `day` variables:

```{r}
fires_by_month <- forestfires %>%
  group_by(month) %>%
  summarize(total_fires = n())

fires_by_month %>% 
  ggplot(aes(x = month, y = total_fires)) +
  geom_col() +
  labs(
    title = "Number of forest fires in data by month",
    y = "Fire count",
    x = "Month"
  )

fires_by_days <- forestfires %>%
  group_by(day) %>%
  summarize(total_fires = n())

fires_by_days %>% 
  ggplot(aes(x = day, y = total_fires)) +
  geom_col() +
  labs(
    title = "Number of forest fires in data by day",
    y = "Fire count",
    x = "Days of the week"
  )
```

The worse months for fires are august and september and most of them start on sundays or mostly in the weekends. 

![fires_per_month](/img/fires_month.png)

![fires_per_day](/img/fire_day.png)

Apart from the summer, I notice a small peak in fire counts in february and march. It's not clear why to me. For the days of the week there is a clear pattern, fires mostly start on the weekend, with a peak on sunday, and are reduced during the week, with the lowest count happening on wednesday, the mid of the week.

### How?

Let's see how the other variables correlate with months.

In order to use `pivot_longer` load `tidyr`

```{r}
> pivoted_fires <- forestfires %>% 
  pivot_longer(cols = c('FFMC', 'DMC', 'DC', 
                        'ISI', 'temp', 'RH', 
                        'wind', 'rain', 'area'), 
                        names_to = 'feature', 
                        values_to = 'condition')

> pivoted_fires %>% ggplot(aes(x = month, y = condition)) + 
                    geom_boxplot() + 
                    facet_wrap(vars(feature), scales = "free_y") + 
                    labs(
                      title = "Variable changes over month", 
                      x = "Month", 
                      y = "Variable value"
                      )
```

![vals_per_month](/img/var_change_per_month.png)

Clearly, temperatures in Portugal during summer months get higher (in august they can get above $30 °$ Celsius). 

A narrower increase over the summer months is that of the `DC` values. This number is related to the drought condition of the soil. During warm months the soil gets drier, hence a fire is easier to start.

The `DMC` index shows a similar behaviour with the `DC`. 

The median vallue of `area` is always very small, but outliers can be very far from the average behaviour during hot months.

`FFMC`, `ISI`, `RH`, `rain` and `wind` don't seems to directly correlate with the fire plot. Well, there are some outliers rains during the months when more fires are registered. This may be a consequence of fires, rather than a causation of them.

I didn't find any significant correlation with the days of the week, as one may expect.

### How bad?

I would like to investigate over the severity of fires. Since no such data is present in the data set, I will use a proxy. I will assume that severity can be described by the `area` the fire has spread over.

```{r}
> pivoted_fires_severity <- forestfires %>% 
  pivot_longer(cols = c('FFMC', 'DMC', 'DC', 
                        'ISI', 'temp', 'RH', 
                        'wind', 'rain'), 
                        names_to = 'feature', 
                        values_to = 'condition')

> pivoted_fires_severity %>% ggplot(aes(x = condition, y = area)) + 
                            geom_point() + 
                            facet_wrap(vars(feature), scales = "free_x") +
                            labs(
                              title = "Correlation between severity (area) and conditions", 
                              x = "Variable value", 
                              y = "Area burned (hectare)"
                              )
```

![severity_proxy](/img/severity_chart.png)

The clearest correlation is that with `rain`: wider areas are impacted when no there is no rain; every time a fire starts while it's raining, the fire burns but a negligible amount of hectares.

By looking at this severity proxy, I see that fires mostly happen when the `ISI` is low - below about $20$ - and/or `FFMC` is high - above $80$. This correlation couldn't be appreciated by looking at correlations with months. However, the `ISI` indicates the spread of the head fire: what I see from this plot is that it's very difficult to start a very big fire - i.e. with `ISI` larger than $20$. The correlation with `FFMC` shows that fires are more likely to happen when less fuel moisture is present on the terrain (this I could understand from [here](https://www.nwcg.gov/publications/pms437/cffdrs/fire-weather-index-system)).

Overall, I can notice two fires which impacted a very big area. I may consider features values at which these happened as rare conditions for extreme phenomena to happen. These are high `DC`, high `FFMC`, no `rain`, low `RH`, high `temp` and average `wind`.

If these two data are not considered, other favourable conditions for severe fires are: high `temp`, strong enough `wind` (it should just be able to move the fire but not too strong to extinguish it) and low relative humidity `RH`.
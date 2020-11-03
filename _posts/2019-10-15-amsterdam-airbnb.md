---
layout: post
title: An Investment in Amsterdam
categories: [project,R]
---

What to recommend when a client wants to invest in an AirBnB property in Amsterdam?




### Initial EDA
Original data was loaded in Excel to gather initial data insights, including:
* Duplicate host_id associated with unique id (property ID) - multiple properties?
* City is not consistent, has Neighborhoods in city column
* Inconsistent naming scheme in state column
* Blanks noted in state column
* Additional insights provided in DataExplorer output, including data types, etc.

Data uploaded to data.world to run against R (using a SQL hook to import), DataExplorer package used to visualize data characteristics (as shown in [this initial EDA](final-project/edaReport_initialdata.pdf). An initial R code was provided to guide this:

```
{
  library(tidyverse)
  library(data.world)
  library(DataExplorer)
  library(Hmisc)
  sql_stmt <- qry_sql('
  select * from unit_1_project_dataset
  ')
  df <- data.world::query(sql_stmt, "ga-data-analytics/project-1-dataset")
  names(df)

  dplyr::filter(df, duplicated(df)) %>% View() # View duplicate rows.
  config <- list("introduce" = list(), "plot_str" = list("type" = "diagonal", "fontSize" = 35, "width" = 1000, "margin" = list("left" = 350, "right" = 250)),
  "plot_missing" = list(),
  "plot_histogram" = list(),
  "plot_qq" = list(sampled_rows = 1000L),
  "plot_bar" = list(),
  "plot_correlation" = list("cor_args" = list("use" = "pairwise.complete.obs")),
  # "plot_prcomp" = list(),
  "plot_boxplot" = list(),
  "plot_scatterplot" = list(sampled_rows = 1000L)
  )
  DataExplorer::create_report(df %>% dplyr::select(1:9, 11:34), y = "price", config=config)
}
```

This exploratory analysis will be used to guide data cleaning.

### Cleaning Methods Used

All data was cleaned in R. The additional provided Translations files were also used to clean and normalize the data.


1. Impute or omit NULL values

The dataset was subsetted to only include rows with NA values (NULLs in R). This allowed manual review to determine best steps to resolve these values.

  Missing values exist for state, beds, bedrooms, bathrooms, zipcode, host response rate, and each of the review scores
  - neighborhood column translated using additional file provided, no NULLS existed
  - city column used translated value, effectively imputing all values to Amsterdam
  - state column NULLs (0.1%) imputed to North Holland per State Translation file, all other rows translated to North Holland
  - zip column NULLs (2.22%) imputed to mode of zip by neighborhood
  - bedrooms (0.18%), beds (0.17%), and bathrooms (0.88%) NULL rows were omitted from the data set, they comprised less than 1% of rows and could not be reasonably imputed - 49 rows affected
  - host response rate (HRR) column NULLs (9.35%) imputed to zero
  - review scores rows removed if all values zero
  - review scores rows containing NaN values also removed, due to effect on dataset; relative low effect of removal on dataset - 14 rows affected, all with 2 or fewer reviews
  - review scores NULLs imputed to mean value of scores if at least one rating existed to retain the mean rating while also giving a more reasonable approximation of the value of these ratings had they been provided
 
 Review scores NULLs existed for over 20% of rows, but not all rows contained NULLs for all review values. Rows with zero reviews were removed from the data set per the given instructions.


2. Remove duplicates

Using the dplyr package in R, duplicate rows were identified. These were manually checked against the dataset in Excel. Once the duplicates were checked and verified, these were eliminated from the dataset. Approximately 20 rows were affected.

Once cleaning was complete, another EDA report was run on the new data. See [clean data EDA report](final-project/edaReport_cleandata.pdf)


### Feature Engineering - in R

1. A zip_no column was created, containing only the numerical portion of the provided zip code. The analysis did not require more granularity than "neighborhood" so specificity here was not needed.
2. host_since_anniversary was split into month and day, with day imputed to the first of the month if null, then combined with existing host_since_year column to generate a single column host_since date in date format. Two null host_since rows were ignored after identification here.
3. Additional columns added to be able to calculate revenue: number of people accommodated, number of guests included, cost per night for max number accomodated, cost to book, estimated number of stays, and estimated total revenue

Full code available [here](final-project/a_dam_airbnb.R)

### Recommendations based on Prompt 2

The data was evaluated categorically by type of property, including subtype (i.e. "Apartment - entire home" and "Apartment - Private Room") as a function of both rating value and count of ratings. Reading the prompt as "highest number of positive reviews" generated "Apartment - Entire home/apt" as the clear leader, followed by . Further analysis is needed to evaluate this more effectively.

Which Property Types receive the most positive reviews?

![Average Reviews by Property Type as a function of Review and Count](https://github.com/Lwillio/ga-adam-airbnb/blob/main/final-project/images/avgRev-propType-reviews.png?raw=true)

Based on the data (and the interpretation at the time of "highest count of high reviews"), **Apartment** listings for the entire apartment received the most positive reviews, followed by **House - Entire** and **Boat - Entire** respectively.

For investment recommendations, Revenue was the primary indicator used to advise the client, both by Property Type and by Neighborhood. 

![Revenue by Property Type](https://github.com/Lwillio/ga-adam-airbnb/blob/main/final-project/images/revenueByPropType.png)

The top three property types to recommend were **Villa**, **Boat**, and **Loft** in that order, based on median revenue. 

![Revenue by Neighborhood](https://github.com/Lwillio/ga-adam-airbnb/blob/main/final-project/images/revenueByNeighborhood.png)

The top three neighborhoods to recommend were, in order; **Center-West**, **Center-East**, and **The Baarsjes** - with Center-West as the clear leader for total revenue. 

Finally, updating the Average Reviews visualization from before to a function of Revenue rather than Count of Reviews, we see **Apartment** as the leader once more, followed by **House - Entire** and **Boat - Entire** more closely tied for second than before.

![Average Revenue by Property Type as function of Revenue](https://github.com/Lwillio/ga-adam-airbnb/blob/main/final-project/images/avgRev-propType-revenue.png)

Combining these data outcomes, a final recommendation was made for investing in a **Boat** in the **Center-West** neighborhood, with the intention of renting the entire property on AirBnB. Revenue was prioritized over reviews to arrive at this recommendation, however, reviews determined the final recommendations from the available options.

_Full Tableau workbook available_ [here](https://public.tableau.com/profile/lg1798#!/vizhome/Project1_672/HMAVGREVPTRT), _please see Metadata at the bottom of the page to navigate between visualizations._


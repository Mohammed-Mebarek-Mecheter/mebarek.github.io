---
weight: 1
title: "Billionaires"
date: 2023-10-12T14:30:28+08:00
lastmod: 
draft: false
author: "Mebarek"
authorLink: "https://www.linkedin.com/in/mohammed-mebarek-mecheter/"
description: ""
images: []
resources:
- name: "featured-image"
  src: "Billionaires.jpg"

categories: ["Projects"]

lightgallery: true

toc:
  auto: false
math:
  enable: true
---

Building off the Forbes World’s Billionaires lists from 1996-2014

<!--more-->


##  Data Cleaning and Preprocessing
-   We started by loading the dataset named `billionaires.csv`, which contains detailed information about billionaires.
-   The dataset includes various columns, such as name, rank, year, company details, demographics, wealth, and more.
-   We performed data cleaning to address missing values, duplicates, and outliers.
-   Columns with missing values were identified and handled as needed.
-   Saved the cleaning data as a dataset named `cleaned_billionaires.csv`
## Wealth Trends Over Time

In this project, we are exploring the wealth trends of billionaires over time using a dataset that includes information from Forbes World's Billionaires lists spanning from 1996 to 2014. The dataset contains various attributes related to billionaires, including their wealth, demographics, and company details. We will document the entire project, step by step.

### Part 1: Total Wealth Over the Years

-   We analyzed the total wealth of billionaires over the years.
    ```python
    # Group the data by 'year' and calculate the total wealth for each year
    yearly_wealth = data.groupby('year')['wealth.worth in billions'].sum().reset_index()
    print(yearly_wealth)
    ```
    ```code
       year  wealth.worth in billions
    0  1996                     934.7
    1  2001                    1710.1
    2  2014                    6407.4
    ```
-   We aggregated the data by year and calculated the total wealth for each year using aggregation functions like sum.
    ```python
    # Create a line chart
    plt.figure(figsize=(10, 6))
    plt.plot(yearly_wealth['year'], yearly_wealth['wealth.worth in billions'], marker='o', linestyle='-')
    plt.title('Total Wealth of Billionaires Over Time')
    plt.xlabel('Year')
    plt.ylabel('Total Wealth (Billions)')
    plt.grid(True)
    
    plt.show()
    ```
{{< figure src="img/V1.png" title="the total wealth for each year" >}}

Key Findings:

-   Total wealth showed a clear upward trend from 1996 to 2014.
-   The line chart revealed a consistent growth pattern with some fluctuations.

### Part 2: Highest and Lowest Wealth Years

-   We identified the highest and lowest years in terms of total billionaire wealth.

Key Findings:

-   The highest year was 2014, with a total billionaire wealth of 6407.4 billion dollars.
-   The lowest year was 1996, with a total wealth of 934.7 billion dollars.

### Part 3: Analysis of Wealth Trends by Geographic Region

-   We analyzed wealth trends by geographic regions and answered questions related to regions that consistently outperform others, regions with unique patterns or fluctuations, and regions with stable or volatile wealth trends.

Key Findings:

-   North America consistently outperformed other regions.
-   Latin America exhibited rapid growth with fluctuations due to the global financial crisis.
-   South Asia showed steady growth with an acceleration in recent years.

Summary Statistics of Total Billionaire Wealth

-   We calculated summary statistics for total billionaire wealth, including mean, median, and standard deviation.

Key Findings:

-   Mean Wealth: 3017.4 billion dollars
-   Median Wealth: 1710.1 billion dollars
-   Standard Deviation of Wealth: 2961.31 billion dollars

Hypothesis Testing

-   We conducted a hypothesis test to determine whether the means of two different years were significantly different.

Key Findings:

-   The null hypothesis was rejected, indicating that the means were significantly different.

Wealth Growth by Region:

-   North America, East Asia, and Europe showed consistent growth.
-   Latin America exhibited rapid growth but faced a decline during the financial crisis.
-   South Asia demonstrated steady growth.
-   Sub-Saharan Africa had slower growth with fluctuations.

Wealth Growth by Sector:

-   Media, real estate, technology, banking, pharmaceuticals, retail, and oil sectors showed significant growth.
-   Some sectors, like hedge funds, faced fluctuations in wealth.

Statistical Analysis:

-   Summary statistics were calculated for regions and sectors to assess wealth growth.

Patterns and Trends:

-   Patterns and trends in wealth growth were identified based on region and sector analysis.

### Conclusion

- This part provides a comprehensive understanding of how wealth among billionaires has evolved over the years, showcasing variations by region and sector. It offers valuable insights into the dynamics of billionaire wealth, highlighting both consistent growth and fluctuations in different areas and industries.

# Self-Made vs. Inherited Wealth Analysis

In this project, we conducted an analysis of the dataset of billionaires, focusing on the classification of billionaires' wealth as self-made or inherited. We also explored whether gender and location play a role in determining whether a billionaire's wealth is self-made or inherited.

### Step 1: Data Preparation

1. We began by loading the dataset, which contains information about billionaires from 1996 to 2014.

2. We filtered the dataset to include only the relevant columns, 'wealth.how.inherited' and 'wealth.how.was founder,' which are essential for categorizing billionaires based on the source of their wealth.

3. A new binary column, 'self_made,' was created to categorize individuals as either self-made or inherited based on the information in the 'wealth.how.inherited' and 'wealth.how.was founder' columns.

### Step 2: Proportion Calculation

4. We calculated the proportion of self-made billionaires and those with inherited wealth in the dataset.

5. The proportion of self-made billionaires was determined by counting the number of individuals categorized as "self-made" and dividing it by the total count of billionaires.

6. The proportion of inherited billionaires was calculated by subtracting the self-made proportion from 1.

7. The results indicated that approximately 64.04% of the billionaires were self-made, while around 35.96% had inherited wealth.

## Gender Analysis

8. To explore the role of gender, we counted the number of male and female billionaires in the dataset.

9. The dataset contains 2,288 male billionaires and 248 female billionaires.

### Step 3: Gender and Location Analysis

10. We grouped the data by gender to further investigate the proportions of self-made and inherited billionaires among males and females.

11. The proportions for each gender were calculated and displayed to examine whether gender plays a role in wealth origin.

### Step 4: Location Analysis

12. We also explored whether location (citizenship) has an impact on the source of wealth for billionaires.

13. The data was grouped by location, and proportions of self-made and inherited billionaires for each location were calculated and analyzed.

### Conclusion

In this analysis, we found that gender and location can play a role in whether a billionaire's wealth is self-made or inherited. The proportions of self-made and inherited billionaires varied among different genders and locations.

This project provides valuable insights into the sources of wealth among billionaires and helps us understand the dynamics of wealth accumulation in the dataset.

- GitHub Repository: https://github.com/Mohammed-Mebarek-Mecheter/Basic-Data-Analysis-Projects/tree/main/Billionaires
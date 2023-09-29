---
weight: 1
title: "European Football Match"
date: 2023-09-28T23:10:01+08:00
lastmod: 2020-03-06T21:29:01+08:00
draft: false
author: "Mebarek"
authorLink: ""
description: "All European football match results since 2005."
images: []
resources:
- name: "All European football match"
  src: "euro.JPG"

tags: ["installation", "configuration"]
categories: ["Projects"]

lightgallery: true

toc:
  auto: false
---

All European football match results since 2005.

<!--more-->


## Match Outcome Analysis

### 1. Data Preparation

- Load the dataset from the provided SQLite database (european_database.sqlite).

    ```python
    import sqlite3

    conn = sqlite3.connect('european_database.sqlite') 
    cursor = conn.cursor()

    cursor.execute("SELECT * FROM matchs LIMIT 10")
    rows = cursor.fetchall()

    for row in rows:
        print(row)

    conn.close()
    ```
The Output loot like this:

```
('B1', '2020-08-08', 'Club Brugge', 'Charleroi', 0.0, 1.0, 'A', 2021)
('B1', '2020-08-08', 'Antwerp', 'Mouscron', 1.0, 1.0, 'D', 2021)
('B1', '2020-08-08', 'Standard', 'Cercle Brugge', 1.0, 0.0, 'H', 2021)
('B1', '2020-08-09', 'St Truiden', 'Gent', 2.0, 1.0, 'H', 2021)
('B1', '2020-08-09', 'Waregem', 'Genk', 1.0, 2.0, 'A', 2021)
('B1', '2020-08-09', 'Mechelen', 'Anderlecht', 2.0, 2.0, 'D', 2021)
('B1', '2020-08-09', 'Kortrijk', 'Waasland-Beveren', 1.0, 3.0, 'A', 2021)
('B1', '2020-08-10', 'Oud-Heverlee Leuven', 'Eupen', 1.0, 1.0, 'D', 2021)
('B1', '2020-08-10', 'Oostende', 'Beerschot VA', 1.0, 2.0, 'A', 2021)
('B1', '2020-08-14', 'Mouscron', 'Mechelen', 0.0, 1.0, 'A', 2021)
```

- Explore the 'matchs' table schema to understand its structure and columns.

    ```python
    import sqlite3

    conn = sqlite3.connect('european_database.sqlite')
    cursor = conn.cursor()

    cursor.execute("PRAGMA table_info(matchs)")
    table_info = cursor.fetchall()

    for column_info in table_info:
        print(f"Column: {column_info[1]}, Type: {column_info[2]}")

    conn.close()
    ```
The Output loot like this:

```
Column Name: Div, Data Type: TEXT
Column Name: Date, Data Type: DATE
Column Name: HomeTeam, Data Type: TEXT
Column Name: AwayTeam, Data Type: TEXT
Column Name: FTHG, Data Type: REAL
Column Name: FTAG, Data Type: REAL
Column Name: FTR, Data Type: TEXT
Column Name: season, Data Type: INTEGER
```

### 2. Data Cleaning

- Check for missing values in the 'FTR' column:

    ```python
    import sqlite3

    conn = sqlite3.connect('european_database.sqlite')
    cursor = conn.cursor()

    cursor.execute("SELECT COUNT(*) FROM matchs WHERE FTR IS NULL") 
    missing_values = cursor.fetchone()[0]

    print(f"Missing values in 'FTR': {missing_values}")

    conn.close()
    ```
The Output loot like this:

```Output
Number of missing values in 'FTR' column: 0
```

- Validate values in the 'FTR' column:

    ```python
    import sqlite3

    conn = sqlite3.connect('european_database.sqlite')
    cursor = conn.cursor()

    cursor.execute("SELECT DISTINCT FTR FROM matchs WHERE FTR NOT IN ('H', 'A', 'D')")
    invalid_values = cursor.fetchall()

    if invalid_values:
        print("Invalid FTR values found:")
        for value in invalid_values:
            print(value[0])
    else:
        print("No invalid values in FTR")

    conn.close()
    ```
The Output loot like this:

```Output
No invalid values found in 'FTR' column.
```

### 3. Data Analysis

- Aggregate data by season and league:

    ```sql
    SELECT
        season,
        Div AS League,
        COUNT(CASE WHEN FTR = 'H' THEN 1 END) AS HomeWins,
        COUNT(CASE WHEN FTR = 'A' THEN 1 END) AS AwayWins,
        COUNT(CASE WHEN FTR = 'D' THEN 1 END) AS Draws
    FROM matchs 
    GROUP BY season, League
    ORDER BY season, League
    ```
The rendered output looks like this:

{{< figure src="2.PNG" title="calculate the distribution of match outcomes across all seasons and leagues" >}}
- SQL query to calculate home win percentages for teams
    ```sql
    SELECT
    HomeTeam AS Team,
    COUNT(CASE WHEN FTR = 'H' THEN 1 END) AS HomeWins,
    COUNT(*) AS TotalMatches,
    (COUNT(CASE WHEN FTR = 'H' THEN 1 END) * 100.0 / COUNT(*)) AS HomeWinPercentage
    FROM matchs
    GROUP BY HomeTeam
    HAVING TotalMatches >= 50  -- Optional: Filter for teams with a minimum number of matches played
    ORDER BY HomeWinPercentage DESC;
    ```
{{< figure src="3.PNG" title="calculate home win percentages for teams" >}}
- SQL query to calculate away win percentages for teams
    ```sql
    SELECT
    AwayTeam AS Team,
    COUNT(CASE WHEN FTR = 'A' THEN 1 END) AS AwayWins,
    COUNT(*) AS TotalMatches,
    (COUNT(CASE WHEN FTR = 'A' THEN 1 END) * 100.0 / COUNT(*)) AS AwayWinPercentage
    FROM matchs
    GROUP BY AwayTeam
    HAVING TotalMatches >= 50  -- Optional: Filter for teams with a minimum number of matches played
    ORDER BY AwayWinPercentage DESC;
    ```
{{< figure src="4.PNG" title="calculate away win percentages for teams" >}}
- SQL query to calculate win percentages for each team over multiple seasons (both home and away)
    ```sql
    SELECT
    Team,
    COUNT(CASE WHEN FTR = 'H' THEN 1 END) AS HomeWins,
    COUNT(CASE WHEN FTR = 'A' THEN 1 END) AS AwayWins,
    COUNT(*) AS TotalMatches,
    (COUNT(CASE WHEN FTR = 'H' THEN 1 END) * 100.0 / COUNT(*)) AS HomeWinPercentage,
    (COUNT(CASE WHEN FTR = 'A' THEN 1 END) * 100.0 / COUNT(*)) AS AwayWinPercentage
    FROM (
    SELECT
        HomeTeam AS Team,
        FTR
    FROM matchs
    UNION ALL
    SELECT
        AwayTeam AS Team,
        FTR
    FROM matchs
    ) AS AllMatches
    GROUP BY Team
    HAVING TotalMatches >= 100  -- Optional: Filter for teams with a minimum number of matches played
    ORDER BY HomeWinPercentage DESC, AwayWinPercentage DESC;
    ```
{{< figure src="1.PNG" title="calculate win percentages for each team over multiple seasons (both home and away)" >}}

### 4. Visualization

- Visualize match outcome trends over seasons for each league.

    ```python
    # Execute the query and fetch the results into a DataFrame
    df = pd.read_sql_query(query, conn)
    # Identify any patterns or trends in match outcomes over the years
    df['TotalMatches'] = df['HomeWins'] + df['AwayWins'] + df['Draws']
    df['HomeWinPercentage'] = (df['HomeWins'] / df['TotalMatches']) * 100
    df['AwayWinPercentage'] = (df['AwayWins'] / df['TotalMatches']) * 100
    df['DrawPercentage'] = (df['Draws'] / df['TotalMatches']) * 100
    # Filter data for a specific league (e.g., 'B1')
    filtered_df = df[df['League'] == 'B1']

    # Create a line chart for Home win percentages
    plt.figure(figsize=(10, 6))
    plt.plot(filtered_df['season'], filtered_df['HomeWinPercentage'], marker='o', linestyle='-', label='Home Win Percentage')
    plt.xlabel('season')
    plt.ylabel('Percentage')
    plt.title('Home Win Percentage Over the Years (League B1)')
    plt.legend()
    plt.grid(True)
    plt.show()
    ```
{{< figure src="V1.png" title="line chart for Home win percentages over the years for a specific league (e.g., 'B1')" >}}

- List teams with highest home and away win percentages.

- Present historical analysis of teams with consistent performance.

- Summarize key insights and findings from the analysis.

## Goal Analysis
- Calculate the average number of goals per match in each league and season:
    ```sql
    SELECT
    season,
    Div AS League,
    AVG(FTHG + FTAG) AS AverageGoalsPerMatch
    FROM matchs
    GROUP BY season, League
    ORDER BY season, League;
    ```
{{< figure src="5.PNG" title="Calculate the average number of goals per match in each league and season" >}}
In this SQL query:
- We select the 'season' and 'Div' columns to group the data by season and league.
- We calculate the average number of goals per match by taking the average of the sum of 'FTHG' (final-time home-team goals) and 'FTAG' (final-time away-team goals).
- We use the GROUP BY clause to group the data by 'season' and 'Div' to calculate the average goals for each league and season.
- Finally, we order the results by 'season' and 'Div' for clarity.
- Calculate the average number of goals scored in matches involving each team::
    ```sql
    SELECT Team,
    AVG(TotalGoals) AS AverageGoalsPerMatch
    FROM (
    SELECT HomeTeam AS Team,
        (FTHG + FTAG) AS TotalGoals
    FROM matchs
    UNION ALL
    SELECT AwayTeam AS Team,
        (FTHG + FTAG) AS TotalGoals
    FROM matchs ) AS AllMatches
    GROUP BY Team
    HAVING COUNT(*) >= 100  -- Optional: Filter for teams involved in a minimum number of matches
    ORDER BY AverageGoalsPerMatch DESC;
    ```
{{< figure src="6.PNG" title="Calculate the average number of goals per match in each league and season" >}}
- Difference in goal statistics between home and away matches:
To determine if there is a significant difference in goal statistics between home and away matches, we can perform a statistical analysis. Specifically, we can conduct a hypothesis test to compare the means of goals scored in home matches and away matches. A commonly used test for this purpose is the independent samples t-test. Here's a step-by-step approach we made:
- Step 1: Prepare two datasets:
    ```python
    import sqlite3

    # Connect to the SQLite database
    conn = sqlite3.connect('european_database.sqlite')
    cursor = conn.cursor()

    # Select goals scored in home matches
    cursor.execute("SELECT FTHG FROM matchs")
    home_goals = [row[0] for row in cursor.fetchall()]

    # Select goals scored in away matches
    cursor.execute("SELECT FTAG FROM matchs")
    away_goals = [row[0] for row in cursor.fetchall()]

    # Close the database connection
    conn.close()
    ```
- Step 2: Hypothesis Testing:
Perform a two-sample independent t-test to compare the means of the two datasets (home goals and away goals). The null hypothesis (H0) is that there is no significant difference in the means, while the alternative hypothesis (H1) is that there is a significant difference.

    ```python
    import scipy.stats as stats

    # Perform the t-test
    t_stat, p_value = stats.ttest_ind(home_goals, away_goals)

    # Set the significance level (alpha)
    alpha = 0.05

    # Check the p-value against alpha
    if p_value < alpha:
        print("Reject the null hypothesis: There is a significant difference in goal statistics between home and away matches.")
    else:
        print("Fail to reject the null hypothesis: There is no significant difference in goal statistics between home and away matches.")
    ```
    The result:
    ```Output
    Reject the null hypothesis: There is a significant difference in goal statistics between home and away matches.
    ```
    Based on the analysis, we have rejected the null hypothesis, indicating that there is a significant difference in goal statistics between home and away matches in our dataset.
    This result suggests that there is a statistically significant distinction between the average number of goals scored in home matches and away matches.
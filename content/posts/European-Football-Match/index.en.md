---
weight: 1
title: "European Football"
date: 2023-09-28T23:30:00+08:00
lastmod: 2023-10-03T22:45:00+08:00
draft: false
author: "Mebarek"
authorLink: "https://www.linkedin.com/in/mohammed-mebarek-mecheter/"
description: ""
images: []
resources:
- name: "featured-image"
  src: "euro.jpg"

categories: ["Projects"]

lightgallery: true

toc:
  auto: false
math:
  enable: true
---

All European football match results since 2005.

<!--more-->

# European Football Analysis
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

{{< figure src="img/2.PNG" title="calculate the distribution of match outcomes across all seasons and leagues" >}}
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
{{< figure src="img/4.PNG" title="calculate away win percentages for teams" >}}
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
{{< figure src="img/1.PNG" title="calculate win percentages for each team over multiple seasons (both home and away)" >}}

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
{{< figure src="img/V1.png" title="line chart for Home win percentages over the years for a specific league (e.g., 'B1')" >}}

- List teams with highest home and away win percentages.

- Present historical analysis of teams with consistent performance.

- Summarize key insights and findings from the analysis.

## Goal Analysis
### 1. Average goals per match
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
{{< figure src="img/5.PNG" title="Calculate the average number of goals per match in each league and season" >}}
In this SQL query:
- We select the 'season' and 'Div' columns to group the data by season and league.
- We calculate the average number of goals per match by taking the average of the sum of 'FTHG' (final-time home-team goals) and 'FTAG' (final-time away-team goals).
- We use the GROUP BY clause to group the data by 'season' and 'Div' to calculate the average goals for each league and season.
- Finally, we order the results by 'season' and 'Div' for clarity.
### 2. Teams scoring in matches
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
{{< figure src="img/6.PNG" title="Calculate the average number of goals per match in each league and season" >}}
### 3. Home vs. Away goals
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
## Team Performance Analysis
### 1. Team ranking by performance
- Ranking teams based on their overall performance, assuming a simple point system:
    ```sql
    WITH TeamPoints AS (
    SELECT HomeTeam AS Team,
        SUM(CASE WHEN FTR = 'H' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS Points
    FROM matchs
    GROUP BY Team
    UNION ALL
    SELECT AwayTeam AS Team,
        SUM(CASE WHEN FTR = 'A' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS Points
    FROM matchs
    GROUP BY Team
    )
    SELECT
    t.Team,
    SUM(tp.Points) AS TotalPoints,
    SUM(t.FTHG) - SUM(t.FTAG) AS GoalDifference
    FROM TeamPoints tp
    JOIN
    (
        SELECT DISTINCT HomeTeam AS Team, FTHG, FTAG
        FROM matchs
        UNION ALL
        SELECT DISTINCT AwayTeam AS Team, FTAG, FTHG
        FROM matchs
    ) t
    ON tp.Team = t.Team
    GROUP BY t.Team
    ORDER BY TotalPoints DESC, GoalDifference DESC;
    ```
The rendered output looks like this:

{{< figure src="img/7.PNG" title="rank teams based on their overall performance, assuming a simple point system" >}}
In this query:
- We first calculate the points earned by each team based on their match results, assuming 3 points for a win, 1 point for a draw, and 0 points for a loss. We use a common table expression (CTE) called TeamPoints for this purpose.
- Then, we calculate the goal difference for each team based on goals scored ('FTHG' for home matches and 'FTAG' for away matches).
- Finally, we join the two sets of calculated data (points and goal differences), calculate the total points, and rank teams based on total points and goal differences.

Visualize the top 10 result:
{{< figure src="img/V2.1.png" title="Top 10 Ranked Teams Based on Total Points" >}}
{{< figure src="img/V2.2.png" title="Top 10 Ranked Teams Based on Goal Difference" >}}

### 2. Evolution performance

- To analyze how the performance of a specific team, such as Real Madrid, has evolved over the years, we can create a line chart that shows relevant performance metrics (e.g., total points or goal difference) for Real Madrid in each season. Here's how we can do it using Python and Matplotlib:

    ```python
    # Define the team name (e.g., 'Real Madrid')
    team_name = 'Real Madrid'

    # Execute an SQL query to retrieve relevant data for Real Madrid
    cursor.execute('''
    SELECT season,
        SUM(CASE WHEN HomeTeam = ? THEN FTHG ELSE FTAG END) - SUM(CASE WHEN AwayTeam = ? THEN FTHG ELSE FTAG END) AS GoalDifference,
        SUM(CASE WHEN HomeTeam = ? AND FTR = 'H' THEN 3 WHEN AwayTeam = ? AND FTR = 'A' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS TotalPoints
    FROM matchs
    WHERE HomeTeam = ? OR AwayTeam = ?
    GROUP BY season
    ORDER BY season
    ''', (team_name, team_name, team_name, team_name, team_name, team_name))

    # Fetch the results
    results = cursor.fetchall()
    ```
- Now, we can create a line chart to visualize how Real Madrid's performance has evolved over the years based on the retrieved data. Here's an example:

    ```python
    # Extract season, goal difference, and total points from the results
    seasons = [row[0] for row in results]
    goal_differences = [row[1] for row in results]
    total_points = [row[2] for row in results]

    # Create a line chart to visualize Real Madrid's goal difference over the years
    plt.figure(figsize=(12, 6))
    plt.plot(seasons, goal_differences, marker='o', linestyle='-', label='Goal Difference')
    plt.xlabel('Season')
    plt.ylabel('Goal Difference')
    plt.title('Real Madrid Performance: Goal Difference Over the Years')
    plt.legend()
    plt.grid(True)
    plt.xticks(rotation=45)  # Rotate x-axis labels for readability

    # Create a line chart to visualize Real Madrid's total points over the years
    plt.figure(figsize=(12, 6))
    plt.plot(seasons, total_points, marker='o', linestyle='-', label='Total Points', color='orange')
    plt.xlabel('Season')
    plt.ylabel('Total Points')
    plt.title('Real Madrid Performance: Total Points Over the Years')
    plt.legend()
    plt.grid(True)
    plt.xticks(rotation=45)

    plt.show()
    ```
{{< figure src="img/V3.1.png" title="Real Madrid Performance: Goal Difference Over the Years" >}}
{{< figure src="img/V3.2.png" title="Real Madrid Performance: Total Points Over the Years" >}}

### 3. Performance changes
- To identify teams that have experienced dramatic changes in performance in specific seasons, you can analyze their point differentials or goal differences between consecutive seasons.

    ```python
    # Define the team name (e.g., 'Real Madrid')
    team_name = 'Real Madrid'
    # Execute an SQL query to retrieve relevant data for the specified team
    cursor.execute('''
    SELECT season,
        SUM(CASE WHEN HomeTeam = ? THEN FTHG ELSE FTAG END) - SUM(CASE WHEN AwayTeam = ? THEN FTHG ELSE FTAG END) AS GoalDifference,
        SUM(CASE WHEN HomeTeam = ? AND FTR = 'H' THEN 3 WHEN AwayTeam = ? AND FTR = 'A' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS TotalPoints
    FROM matchs
    WHERE (HomeTeam = ? OR AwayTeam = ?)
    GROUP BY season
    ORDER BY season
    ''', (team_name, team_name, team_name, team_name, team_name, team_name))

    # Fetch the results
    results = cursor.fetchall()
    #%%
    # Extract season and goal difference from the results
    seasons = [row[0] for row in results]
    goal_differences = [row[1] for row in results]

    # Calculate changes in goal difference between consecutive seasons
    goal_difference_changes = [goal_differences[i] - goal_differences[i - 1] for i in range(1, le(goal_differences))]

    # Identify seasons with dramatic changes in goal difference
    dramatic_changes = []
    for i, change in enumerate(goal_difference_changes):
        if abs(change) >= 20:  # Adjust the threshold as needed
            dramatic_changes.append((seasons[i], seasons[i + 1], change))

    # Display seasons with dramatic changes in performance
    for change in dramatic_changes:
        print(f"Season {change[0]} to {change[1]}: Change in Goal Difference = {change[2]}")
    ```
- The result:
    ```
    Season 2007 to 2008: Change in Goal Difference = 22.0
    Season 2009 to 2010: Change in Goal Difference = 36.0
    Season 2011 to 2012: Change in Goal Difference = 20.0
    Season 2012 to 2013: Change in Goal Difference = -28.0
    Season 2018 to 2019: Change in Goal Difference = -33.0
    Season 2019 to 2020: Change in Goal Difference = 28.0
    ```

## Seasonal Trends
### 1. Team Performances
- Step 1: Define the Team (e.g., Real Madrid)

    ```python
    # Define the team name (e.g., 'Real Madrid')
    team_name = 'Real Madrid'
    ```
- Step 2: Retrieve Data for the Team

    ```python
    import sqlite3

    # Connect to the SQLite database
    conn = sqlite3.connect('european_database.sqlite')
    cursor = conn.cursor()

    # Execute an SQL query to retrieve relevant data for the specified team
    cursor.execute('''
    SELECT season, strftime('%Y-%m', Date) AS Month,
        SUM(CASE WHEN HomeTeam = ? THEN FTHG ELSE FTAG END) - SUM(CASE WHEN AwayTeam = ? THEN FTHG ELSE FTAG END) AS GoalDifference,
        SUM(CASE WHEN HomeTeam = ? AND FTR = 'H' THEN 3 WHEN AwayTeam = ? AND FTR = 'A' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS TotalPoints
    FROM matchs
    WHERE (HomeTeam = ? OR AwayTeam = ?)
    GROUP BY season, Month
    ORDER BY season, Month
    ''', (team_name, team_name, team_name, team_name, team_name, team_name))

    # Fetch the results
    results = cursor.fetchall()
    results
    ```

    ```Output
    [(2006, '2005-08', 1.0, 3),
    (2006, '2005-09', 3.0, 6),
    (2006, '2005-10', 6.0, 9),
    (2006, '2005-11', -2.0, 4),
    ...
     (2021, '2021-02', 6.0, 12),
    (2021, '2021-03', 3.0, 8),
    (2021, '2021-04', 6.0, 11),
    (2021, '2021-05', 7.0, 13)]
    ```

- Step 3: Analyze Seasonal Trends

    ```python
    import matplotlib.pyplot as plt

    # Extract season, month, and performance metrics from the results
    seasons = [row[0] for row in results]
    months = [row[1] for row in results]
    goal_differences = [row[2] for row in results]
    total_points = [row[3] for row in results]

    # Create line charts to visualize the team's performance metrics over time
    plt.figure(figsize=(12, 6))
    plt.plot(seasons, goal_differences, marker='o', linestyle='-', label='Goal Difference')
    plt.xlabel('Season')
    plt.ylabel('Goal Difference')
    plt.title(f'{team_name} Performance: Goal Difference Over Time')
    plt.legend()
    plt.grid(True)
    plt.xticks(rotation=45)  # Rotate x-axis labels for readability
    plt.show()

    plt.figure(figsize=(12, 6))
    plt.plot(seasons, total_points, marker='o', linestyle='-', label='Total Points', color='orange')
    plt.xlabel('Season')
    plt.ylabel('Total Points')
    plt.title(f'{team_name} Performance: Total Points Over Time')
    plt.legend()
    plt.grid(True)
    plt.xticks(rotation=45)
    plt.show()
    ```
{{< figure src="img/V4.1.png" title="Team Performance: Goal Difference Over Time" >}}
{{< figure src="img/V4.2.png" title="Team Performance: Total Points Over Time" >}}

 ### 2. Yearly Performance
 - Step 1: Define Performance Metrics

    ```python
    # Define the performance metric to analyze (e.g., 'TotalPoints' or 'GoalDifference')
    performance_metric = 'TotalPoints'
    ```
- Step 2: Retrieve Data for All Teams

    ```python
    # Execute an SQL query to retrieve relevant data for all teams and seasons
    cursor.execute('''
    SELECT
        season,
        HomeTeam AS Team,
        SUM(CASE WHEN FTR = 'H' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS PointsHome,
        SUM(CASE WHEN FTR = 'H' THEN FTHG ELSE 0 END) AS GoalsForHome,
        SUM(CASE WHEN FTR = 'H' THEN FTAG ELSE 0 END) AS GoalsAgainstHome,
        SUM(CASE WHEN FTR = 'A' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS PointsAway,
        SUM(CASE WHEN FTR = 'A' THEN FTAG ELSE 0 END) AS GoalsForAway,
        SUM(CASE WHEN FTR = 'A' THEN FTHG ELSE 0 END) AS GoalsAgainstAway
    FROM
        matchs
    GROUP BY
        season, Team
    ORDER BY
        season, Team
    ''')

    # Fetch the results
    results = cursor.fetchall()
    ```
- Step 3: Analyze Consistent Performance

    ```python
    # Extract season, team, and performance metrics from the results
    seasons = [row[0] for row in results]
    teams = [row[1] for row in results]
    points_home = [row[2] for row in results]
    goals_for_home = [row[3] for row in results]
    goals_against_home = [row[4] for row in results]
    points_away = [row[5] for row in results]
    goals_for_away = [row[6] for row in results]
    goals_against_away = [row[7] for row in results]

    # Calculate the total points and goal differences for each team in each season
    if performance_metric == 'TotalPoints':
        total_points = [points_home[i] + points_away[i] for i in range(len(points_home))]
        performance_values = total_points
    else:
        goal_differences = [(goals_for_home[i] + goals_for_away[i]) - (goals_against_home[i] + goals_against_away[i]) for i in range(len(goals_for_home))]
        performance_values = goal_differences

    # Identify teams that consistently perform better or worse
    consistent_teams = {}
    for i, team in enumerate(teams):
        if team not in consistent_teams:
            consistent_teams[team] = {'Seasons': [seasons[i]], 'PerformanceValues': [performance_values[i]]}
        else:
            consistent_teams[team]['Seasons'].append(seasons[i])
            consistent_teams[team]['PerformanceValues'].append(performance_values[i])

    # Analyze and print teams with consistent better or worse performance
    for team, data in consistent_teams.items():
        min_performance = min(data['PerformanceValues'])
        max_performance = max(data['PerformanceValues'])
        if min_performance < max_performance:
            print(f"{team}: Consistently performs better with a minimum {performance_metric} of {min_performance} and maximum {performance_metric} of {max_performance}.")
        elif min_performance > max_performance:
            print(f"{team}: Consistently performs worse with a minimum {performance_metric} of {min_performance} and maximum {performance_metric} of {max_performance}.")
        else:
            print(f"{team}: Performance varies across seasons with a consistent {performance_metric} of {min_performance}.")
    ```
{{< figure src="img/V4.3.PNG" title="Analyzing multi-season team performance trends." >}}

## Home Advantage Analysis
### 1. Investigating "home advantage"
- Step 1: Define Performance Metrics

    ```python
    # Define the performance metric to analyze (e.g., 'TotalPoints', 'GoalsScored', 'GoalsConceded')
    performance_metric = 'TotalPoints'
    ```
- Step 2: Retrieve Data for All Matches

    ```python
    # Execute an SQL query to retrieve relevant data for all matches and venues
    cursor.execute('''
    SELECT HomeTeam AS Team,
        SUM(CASE WHEN FTR = 'H' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS PointsHome,
        SUM(FTHG) AS GoalsScoredHome,
        SUM(FTAG) AS GoalsConcededHome
    FROM matchs
    GROUP BY Team
    ORDER BY Team
    ''')
    # Fetch the results for matches played at home
    home_results = cursor.fetchall()

    # Execute another SQL query to retrieve relevant data for all matches and venues
    cursor.execute('''
    SELECT AwayTeam AS Team,
        SUM(CASE WHEN FTR = 'A' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS PointsAway,
        SUM(FTAG) AS GoalsScoredAway,
        SUM(FTHG) AS GoalsConcededAway
    FROM matchs
    GROUP BY Team
    ORDER BY Team
    ''')

    # Fetch the results for matches played away
    away_results = cursor.fetchall()
    ```
- Step 3: Calculate Home Advantage Metrics

    ```python
    # Create dictionaries to store home and away performance metrics for each team
    home_performance = {row[0]: (row[1], row[2], row[3]) for row in home_results}
    away_performance = {row[0]: (row[1], row[2], row[3]) for row in away_results}

    # Calculate home advantage metrics for each team
    home_advantage = {}
    for team in home_performance:
        if team in away_performance:
            home_points, home_goals_scored, home_goals_conceded = home_performance[team]
            away_points, away_goals_scored, away_goals_conceded = away_performance[team]

            if performance_metric == 'TotalPoints':
                home_advantage[team] = home_points - away_points
            elif performance_metric == 'GoalsScored':
                home_advantage[team] = home_goals_scored - away_goals_scored
            elif performance_metric == 'GoalsConceded':
                home_advantage[team] = away_goals_conceded - home_goals_conceded

    # Sort teams by home advantage
    sorted_teams = sorted(home_advantage.items(), key=lambda x: x[1], reverse=True)

    # Display teams with the strongest home advantage
    print(f"Teams with the strongest home advantage in terms of {performance_metric}:")
    for team, advantage in sorted_teams[:10]:  # Display the top 10 teams
        print(f"{team}: Home Advantage = {advantage}")
    ```
{{< figure src="img/V5.1.PNG" title="Evaluating home advantage for each team." >}}
### 2. Examining seasonal variations
- Step 1: Retrieve Data for All Matches

    ```python
    # Execute an SQL query to retrieve relevant data for all seasons and venues
    cursor.execute('''
    SELECT season,
        HomeTeam AS Team,
        SUM(CASE WHEN FTR = 'H' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS PointsHome
    FROM matchs
    GROUP BY season, Team
    ORDER BY season, Team
    ''')

    # Fetch the results for matches played at home
    home_results = cursor.fetchall()

    # Execute another SQL query to retrieve relevant data for all seasons and venues
    cursor.execute('''
    SELECT season,
        AwayTeam AS Team,
        SUM(CASE WHEN FTR = 'A' THEN 3 WHEN FTR = 'D' THEN 1 ELSE 0 END) AS PointsAway
    FROM matchs
    GROUP BY season, Team
    ORDER BY season, Team
    ''')

    # Fetch the results for matches played away
    away_results = cursor.fetchall()
    ```
- Step 2: Calculate Seasonal Home Advantage Metrics

    ```python
    # Create dictionaries to store home and away performance metrics for each team by season
    home_performance = {(row[0], row[1]): row[2] for row in home_results}
    away_performance = {(row[0], row[1]): row[2] for row in away_results}

    # Calculate seasonal home advantage metrics for each team
    seasonal_home_advantage = {}
    for season, team in home_performance:
        if (season, team) in away_performance:
            home_points = home_performance[(season, team)]
            away_points = away_performance[(season, team)]

            if performance_metric == 'TotalPoints':
                seasonal_home_advantage[(season, team)] = home_points - away_points
    ```
- Step 3: Visualize Home Advantage Across Seasons

    ```python
    # Extract unique seasons for analysis
    unique_seasons = sorted(set(season for season, _ in seasonal_home_advantage.keys()))

    # Calculate average home advantage per season
    average_home_advantage = []
    for season in unique_seasons:
        season_home_advantages = [advantage for (s, _), advantage in seasonal_home_advantage.items() if s == season]
        average_home_advantage.append(sum(season_home_advantages) / len(season_home_advantages))

    # Create a line chart to visualize seasonal home advantage
    plt.figure(figsize=(12, 6))
    plt.plot(unique_seasons, average_home_advantage, marker='o', linestyle='-', label=f'Average Home Advantage ({performance_metric})')
    plt.xlabel('Season')
    plt.ylabel('Average Home Advantage')
    plt.title(f'Seasonal Home Advantage Variation ({performance_metric})')
    plt.legend()
    plt.grid(True)
    plt.xticks(rotation=45)
    plt.show()
    ```
{{< figure src="img/V5.2.PNG" title="Assessing and visualizing seasonal home advantage metrics for teams." >}}

## Country Comparison Analysis
### 1. Outcomes and Goals
- Step 1: Retrieve Data for Teams and Matches

    ```python
    import sqlite3

    # Connect to the SQLite database
    conn = sqlite3.connect('european_database.sqlite')
    cursor = conn.cursor()

    # Execute an SQL query to join the 'matchs' and 'divisions' tables to retrieve relevant data
    cursor.execute('''
    SELECT d.country AS Country,
        m.HomeTeam AS Team,
        AVG(CASE WHEN m.FTR = 'H' THEN 3 WHEN m.FTR = 'D' THEN 1 ELSE 0 END) AS AveragePoints,
        AVG(m.FTHG) AS AverageGoalsScored,
        AVG(m.FTAG) AS AverageGoalsConceded
    FROM matchs AS m
    JOIN divisions AS d
    ON m.Div = d.division
    GROUP BY d.country, m.HomeTeam
    ORDER BY Country, Team
    ''')

    # Fetch the results
    results = cursor.fetchall()
    ```
- Step 2: Analyze and Visualize Data by Country/League

    ```python
    import pandas as pd
    import seaborn as sns
    import matplotlib.pyplot as plt

    # Convert the results to a pandas DataFrame for easier manipulation and visualization
    df = pd.DataFrame(results, columns=['Country', 'Team', 'AveragePoints', 'AverageGoalsScored', 'AverageGoalsConceded'])

    # Plot the data using seaborn
    plt.figure(figsize=(12, 6))
    sns.set(style="whitegrid")
    sns.scatterplot(data=df, x='AverageGoalsScored', y='AverageGoalsConceded', hue='Country', s=100)
    plt.xlabel('Average Goals Scored')
    plt.ylabel('Average Goals Conceded')
    plt.title('Comparison of Teams by Country/League (Goals Scored vs. Goals Conceded)')
    plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left')
    plt.show()

    plt.figure(figsize=(12, 6))
    sns.set(style="whitegrid")
    sns.scatterplot(data=df, x='AveragePoints', y='AverageGoalsScored', hue='Country', s=100)
    plt.xlabel('Average Points Earned')
    plt.ylabel('Average Goals Scored')
    plt.title('Comparison of Teams by Country/League (Points vs. Goals Scored)')
    plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left')
    plt.show()
    ```
{{< figure src="img/V6.1.PNG" title="Comparison of Teams by Country/League (Goals Scored vs. Goals Conceded)." >}}
{{< figure src="img/V6.2.PNG" title="Comparison of Teams by Country/League (Points vs. Goals Scored)." >}}
### 3. Competitive Leagues
- Step 1: Retrieve Data for Leagues

    ```python
    # Execute an SQL query to join the 'matchs' and 'divisions' tables to retrieve relevant data
    cursor.execute('''
    SELECT
        d.country AS Country,
        d.name AS League,
        COUNT(*) AS TotalMatches,
        AVG(ABS(FTHG - FTAG)) AS AverageGoalDifference,
        AVG(CASE WHEN FTR = 'D' THEN 1 ELSE 0 END) AS DrawPercentage
    FROM matchs AS m
    JOIN divisions AS d
    ON m.Div = d.division
    GROUP BY d.country, d.name
    ORDER BY Country, League
    ''')

    # Fetch the results
    results = cursor.fetchall()
    ```
- Step 2: Analyze and Identify Leagues with Competitive Matches

    ```python
    # Convert the results to a pandas DataFrame for easier manipulation and visualization
    df = pd.DataFrame(results, columns=['Country', 'League', 'TotalMatches', 'AverageGoalDifference', 'DrawPercentage'])

    # Plot the data using seaborn
    plt.figure(figsize=(12, 6))
    sns.set(style="whitegrid")
    sns.scatterplot(data=df, x='AverageGoalDifference', y='DrawPercentage', hue='Country', s=100)
    plt.xlabel('Average Goal Difference')
    plt.ylabel('Draw Percentage')
    plt.title('Comparison of Leagues by Competitiveness (Goal Difference vs. Draw Percentage)')
    plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left')
    plt.show()
    ```
{{< figure src="img/V6.3.PNG" title="Comparison of Leagues by Competitiveness (Goal Difference vs. Draw Percentage)." >}}

## Time Series Analysis
- Step 1: Retrieve Historical Data
    ```python
    import sqlite3
    import pandas as pd

    # Connect to the SQLite database
    conn = sqlite3.connect('european_database.sqlite')

    # Query to retrieve historical data for match outcomes and goal statistics
    query = '''
    SELECT
        strftime('%Y', Date) AS Year,
        COUNT(*) AS TotalMatches,
        AVG(CASE WHEN FTR = 'H' THEN 1 ELSE 0 END) AS HomeWinPercentage,
        AVG(CASE WHEN FTR = 'A' THEN 1 ELSE 0 END) AS AwayWinPercentage,
        AVG(CASE WHEN FTR = 'D' THEN 1 ELSE 0 END) AS DrawPercentage,
        AVG(FTHG) AS AverageHomeGoals,
        AVG(FTAG) AS AverageAwayGoals
    FROM matchs
    GROUP BY Year
    ORDER BY Year
    '''

    # Fetch the data into a pandas DataFrame
    df = pd.read_sql_query(query, conn)
    ```
- Step 2: Analyze and Visualize Trends

    ```python
    import matplotlib.pyplot as plt

    # Convert the 'Year' column to datetime format
    df['Year'] = pd.to_datetime(df['Year'], format='%Y')

    # Create subplots for different metrics
    fig, axes = plt.subplots(nrows=2, ncols=2, figsize=(14, 10))
    fig.suptitle('Time Series Analysis of Match Outcomes and Goal Statistics Over the Years', fontsize=16)

    # Plot Total Matches Over the Years
    axes[0, 0].plot(df['Year'], df['TotalMatches'], marker='o', linestyle='-')
    axes[0, 0].set_title('Total Matches Over the Years')
    axes[0, 0].set_xlabel('Year')
    axes[0, 0].set_ylabel('Total Matches')

    # Plot Home Win Percentage Over the Years
    axes[0, 1].plot(df['Year'], df['HomeWinPercentage'], marker='o', linestyle='-', color='orange')
    axes[0, 1].set_title('Home Win Percentage Over the Years')
    axes[0, 1].set_xlabel('Year')
    axes[0, 1].set_ylabel('Home Win Percentage')

    # Plot Away Win Percentage Over the Years
    axes[1, 0].plot(df['Year'], df['AwayWinPercentage'], marker='o', linestyle='-', color='green')
    axes[1, 0].set_title('Away Win Percentage Over the Years')
    axes[1, 0].set_xlabel('Year')
    axes[1, 0].set_ylabel('Away Win Percentage')

    # Plot Draw Percentage Over the Years
    axes[1, 1].plot(df['Year'], df['DrawPercentage'], marker='o', linestyle='-', color='red')
    axes[1, 1].set_title('Draw Percentage Over the Years')
    axes[1, 1].set_xlabel('Year')
    axes[1, 1].set_ylabel('Draw Percentage')

    # Adjust layout
    plt.tight_layout(rect=[0, 0, 1, 0.95])

    # Show the plots
    plt.show()
    ```
{{< figure src="img/V7.png" title="Visualizing historical match outcomes and goal statistics trends." >}}

- GitHub Repository: https://github.com/Mohammed-Mebarek-Mecheter/European-Football-Analysis
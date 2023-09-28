---
weight: 1
title: "Theme Documentation - Basics"
date: 2023-09-28T23:10:01+08:00
lastmod: 2020-03-06T21:29:01+08:00
draft: false
author: "Mebarek"
authorLink: "https://dillonzq.com"
description: "All European football match results since 2005."
images: []
resources:
- name: "All European football math"
  src: "All European football math.jpg"

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

- Identify trends in match outcomes over the years for each league.

- Calculate home and away win percentages for each team to find teams with highest win percentages.

- Analyze team performance over multiple seasons to identify teams that consistently perform well or poorly.

### 4. Presentation

- Visualize match outcome trends over seasons for each league.

- List teams with highest home and away win percentages.

- Present historical analysis of teams with consistent performance.

- Summarize key insights and findings from the analysis.
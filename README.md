# Mobile-Game-Retention-Project
We've been hired by a mobile game company. Like most mobile games, this game has a store where players can buy a vast array of different items. Matches are composed of two players going head-to-head against each other. These two facts mean that there is a rich store of four tables:

As it is the game's one-year anniversary, my manager has asked me and one other team member to investigate player retention. Of specific interest is counting rolling 30-day retention and expressing it as a fraction of the total playerbase at the time. The tools we used are BigQuery and Google Sheets.

To address the question, we used the two following tables for our investigation:

Match information, including the players who matched against each other, and the outcome
Player information, including information like the player's age and when they joined
The SQL table should be including the folloing columns:

The day in question
The total number of players who joined that day
Of the players who joined, how many were retained
The fractional retention (the third column divided by the second column).
Q1: Is 30-day rolling retention increasing or decreasing over the lifecycle of the game?
![image](https://user-images.githubusercontent.com/94933743/155800230-f781d605-d61e-48c3-b59e-376e43152532.png)
We excluded the last 30 days of the year from our analysis because those who joined in the previous 30 days would have automatically been categorized as Not-Retained. As a result, notably, we saw a dramatic decline at the end of the year. Therefore, we only choose the data from Day 1 to Day 335 for this project.

Given the data above, we observed that from the trendline, overall, the retention rate was relatively consistent throughout the year(Average 65.46%). From a growth perspective, while the retention rate is stable in the trendline, the growth hasn't significantly increased since the Q1. There appear to be a few noticeable peaks in the retained player count (e.g. day 55 & 277), but it quickly adjusted back to the average line. The result may be a signal to remind us, perhaps, it's time to reshift our foucs back on Long-term retention and YOY growth strategies.
Q2: Do players with rolling 30-day retention come from specific regions?
Regions with Retention
![image](https://user-images.githubusercontent.com/94933743/155808998-68e3f065-0f0d-4e1a-a001-6256c468241e.png)

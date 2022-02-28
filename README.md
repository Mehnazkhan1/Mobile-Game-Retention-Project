# Mobile-Game-Retention-Project
We've been hired by a mobile game company. Like most mobile games, this game has a store where players can buy a vast array of different items. Matches are composed of two players going head-to-head against each other. These two facts mean that there is a rich store of four tables:

As it is the game's one-year anniversary, my manager has asked me and one other team member to investigate player retention. Of specific interest is counting rolling 30-day retention and expressing it as a fraction of the total playerbase at the time. The tools we used are BigQuery and Google Sheets.

To address the question, we used the two following tables for our investigation:

Match information, including the players who matched against each other, and the outcome
Player information, including information like the player's age and when they joined
The SQL table should be including the folloing columns:

1-The day in question
2-The total number of players who joined that day
3-Of the players who joined, how many were retained
4-The fractional retention (the third column divided by the second column).

# Q1: Is 30-day rolling retention increasing or decreasing over the lifecycle of the game?
Querry:
SELECT
    joined Day_joined,
    COUNT(player_id) Players_joined,
    SUM(Retention_Status) Players_retained,
    ROUND((SUM(Retention_Status)/COUNT(player_id)),2) AS fraction_retention,
FROM 
    (SELECT
    p.player_id,
    p.joined,
    CASE
    WHEN (MAX(M.day) - (p.joined)) >= 30 THEN 1
    ELSE
    0
    END  Retention_Status
 FROM
    `myjunoproject.cohort2_Mysql_project.matches_info` m
JOIN
    `cohort2_Mysql_project.player_info` p
 ON
    p.player_id = M.player_id
GROUP BY
    p.joined,
    p.player_id) AS retention_status_players
GROUP BY
    joined;
  
We excluded the last 30 days of the year from our analysis because those who joined in the previous 30 days would have automatically been categorized as Not-Retained. As a result, notably, we saw a dramatic decline at the end of the year. Therefore, we only choose the data from Day 1 to Day 335 for this project.
Given the data above, we observed that from the trendline, overall, the retention rate was relatively consistent throughout the year(Average 65.46%). From a growth perspective, while the retention rate is stable in the trendline, the growth hasn't significantly increased since the Q1. There appear to be a few noticeable peaks in the retained player count (e.g. day 55 & 277), but it quickly adjusted back to the average line. The result may be a signal to remind us, perhaps, it's time to reshift our foucs back on Long-term retention and YOY growth strategies.

# Q2: Do players with rolling 30-day retention come from specific regions?
In this analsis it looks like this game is equally famous in all the locations because we could see a huge difference in numbers.
Location with Retention
![image](https://user-images.githubusercontent.com/94933743/155808998-68e3f065-0f0d-4e1a-a001-6256c468241e.png)
SELECT
      DISTINCT p.location AS location,
      COUNT(DISTINCT p.player_id) AS players,
      CASE
      WHEN MAX(m.day)-MIN(p.joined) > 30 THEN 1
      ELSE
      0
END
      AS num_retained
FROM
     `myjunoproject.cohort2_Mysql_project.matches_info` m
JOIN
     `cohort2_Mysql_project.player_info` p
ON
      p.player_id = M.player_id
GROUP BY
      p.location
ORDER BY
      players DESC;
  
 # Q 3:Do players with rolling 30-day retention spend more?
Querry
----
SELECT ROUND(SUM(sum_price),2) AS sum_total_spend,
       retention_table.retention_status
FROM
       (SELECT 
       pr.player_id,
       Round(SUM(i.price),2) as sum_price
 FROM
       `myjunoproject.cohort2_Mysql_project.purchase_info`  pr
 JOIN
       `myjunoproject.cohort2_Mysql_project.item_info`   i
 ON
        pr.item_id = i.item_id
 GROUP BY 
        pr.player_id) total_spends

 JOIN
    (SELECT 
    player_id,
    CASE WHEN max_days >= joined + 29 THEN 1 ELSE 0  END retention_status

FROM
    (SELECT 
    p.player_id,
    Max(m.day) AS max_days,
    p.joined
FROM 
   `myjunoproject.cohort2_Mysql_project.matches_info` m
JOIN 
   `cohort2_Mysql_project.player_info` p
 ON
    p.player_id = M.player_id
    GROUP BY p.player_id,p.joined) AS max_retained_days
    ORDER BY 2 desc) retention_table
    ON
    total_spends.player_id = retention_table.player_id
 GROUP BY retention_table.retention_status;
   
 
 # Q4: What is the average age of player for this game?
  The average age of playe is in 20s.
  Querry
  SELECT
      DISTINCT location,
      ROUND(avg(age) over (partition by location),2) as max_age_player,
      COUNT(player_id) over (partition by location) total_player,

  FROM
      `cohort2_Mysql_project.player_info`
       ORDER BY 
       location;
     
# Q5:What s the total number of wins & loss in retained players & not retained players?
We came to know that total wins of retained players are 330047 & total wins of not retained players are 79459.
Total number of loss in retained player are 329708 and not retained players are 79759.
Querry
SELECT 
COUNT(player_id) total_players, 
SUM(total_analsis.total_wins) total_wins,
SUM(total_analsis.total_losses) total_loss,
retention_status------not retained people win & loss
From
(SELECT 
win_loss_table.player_id,
win_loss_table.total_wins,
win_loss_table.total_losses,
retention_table.retention_status

FROM

(SELECT 
win_matches.player_id,
win_matches.total_wins,
loss_matches.total_losses

 FROM 
 ---this one is for total wins
(SELECT player_id,
Count(outcome) AS total_wins
 FROM `myjunoproject.cohort2_Mysql_project.matches_info` 
 Where outcome = 'win'
 Group by player_id) AS win_matches
JOIN
---this one is for total loss
 (SELECT player_id,
Count(outcome) AS total_losses
 FROM `myjunoproject.cohort2_Mysql_project.matches_info` 
 Where outcome = 'loss'
 Group by player_id) AS loss_matches
 ON win_matches.player_id = loss_matches.player_id) win_loss_table

JOIN 
---retention table
(SELECT 
player_id,
CASE WHEN max_days >= joined + 29 THEN 1 ELSE 0  END retention_status

FROM
(SELECT 
p.player_id,
Max(m.day) AS max_days,
p.joined
FROM 
`myjunoproject.cohort2_Mysql_project.matches_info` m
JOIN 
 `cohort2_Mysql_project.player_info` p
 ON
    p.player_id = M.player_id
    GROUP BY p.player_id,p.joined) AS max_retained_days
    ORDER BY 2 desc ) AS retention_table

ON win_loss_table.player_id = retention_table.player_id
Order by 3 desc ) total_analsis
Group By retention_status;


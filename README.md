# Mobile-Game-Retention-Project
![image](https://user-images.githubusercontent.com/94933743/156075183-1cecad7c-d679-4578-a316-1102c9ed1d6c.png)

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

![Retention Increase Or Decrease Status ](https://user-images.githubusercontent.com/94933743/156066008-40ee3c85-2e56-4326-b7cc-c617d585d102.png)

Querry:
```
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
```
  
We excluded the last 30 days of the year from our analysis because those who joined in the previous 30 days would have automatically been categorized as Not-Retained. As a result, notably, we saw a dramatic decline at the end of the year. Therefore, we only choose the data from Day 1 to Day 335 for this project.
Given the data above, we observed that from the trendline, overall, the retention rate was relatively consistent throughout the year(Average 65.46%). From a growth perspective, while the retention rate is stable in the trendline, the growth hasn't significantly increased since the Q1. There appear to be a few noticeable peaks in the retained player count (e.g. day 55 & 277), but it quickly adjusted back to the average line. The result may be a signal to remind us, perhaps, it's time to reshift our foucs back on Long-term retention and YOY growth strategies.

# Q2: Do players with rolling 30-day retention come from specific regions?

![image](https://user-images.githubusercontent.com/94933743/156236702-358f7e86-797b-4c72-b1d4-5dac194d45d5.png)


```
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
```  

As indicated by the visualizations provided, we have a total of 28859 people with rolling 30-day retention. The highest retention player number is located in South America(4863), followed by North America(4855) and Oceania(4832).

In this analysis it looks like this game is equally famous in all the locations since the percentage of the differences between the highest player region amount and the lowest is less than 3%..

To sum up the indication, the region doesn't matter. Although there are some days we could see total retained player's differences in the year, the result would always regress to the reference value if we look at the pictures from a quarter to quarter basis.

 # Q 3:Do players with rolling 30-day retention spend more?
 According to our analysis people who retained have spend more on game as compare to those players who have not retained.
 ![Total Spend of Retained   Non Retained Players](https://user-images.githubusercontent.com/94933743/156071986-56cb5e16-2410-423e-8ff1-0261d6a681ab.png)

Querry
----
```
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
```   
 
 # Q4: What is the average age of player for this game?
  The average age of players are in 20s.
  ![SUM of total_players and AVERAGE of age](https://user-images.githubusercontent.com/94933743/156072688-b5fa4231-bf2d-4ba5-9780-bc52e5594a2f.png)
 Querry
 ```
  SELECT
      DISTINCT location,
      ROUND(avg(age) over (partition by location),2) as max_age_player,
      COUNT(player_id) over (partition by location) total_player,

  FROM
      `cohort2_Mysql_project.player_info`
       ORDER BY 
       location;
```     
# Q5:What s the total number of wins & loss in retained players & not retained players?
There is not a huge difference between the loss & win matched. But retained peole have played more matched so their win loss ratio is more than the players who have not retained.

|total wins of retained players 330047|	

|total loss of retained players 329708 |

|total wins of non-retained players 79459|

|total loss of non-retained players 79759|

![Total Wins   Loss of Retained   Non-Retained layers](https://user-images.githubusercontent.com/94933743/156071449-46725b28-8867-4184-b716-51e03d85c35c.png)

We came to know that total wins of retained players are 330047 & total wins of not retained players are 79459.
Total number of loss in retained player are 329708 and not retained players are 79759.

Querry
```
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
```

## Conclusion
In short sentences, the datasets provided us with some threads that was able to help us find the trend. While there is much more we could further investigate, we were able to tell the retention rate is quite stable throughout the year. However, it isn't necessarily a negative indication, but it's worth focusing on long-term strategies. Also, we saw players are distributed to all regions, which indicates that our marketing strategies is equally launched by geography. Last but not least, in our findings, people with a higher winning rate are more willing to be retained in this game. If we want to increase retention, we could perhaps redesign the winning criteria to increase the retention rate.
Also we can see that game is popular among all the locations and people who play it more are in their 20's. We had a limited data so our analysis are limited. Retention status is consistant but at the end it has a big drop. Which could be caused by different elements either game is too easy for the players or too too tough or their is not enough interesting elements in the game for certain age groups. So people join and get bored of it.


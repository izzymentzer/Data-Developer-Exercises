# Data-Developer-Exercises
# First open SQLite, here is my code for the first three problems
# Problem 1
.open main
.output problem1.csv
SELECT 
    playerID, 
    name, 
year,
WAR,
    SUM(CAST(WAR AS FLOAT)) AS combined_WAR
FROM WAR
WHERE (year = '2002' OR year = '2003') 
GROUP BY playerID, name
HAVING (CASE WHEN year = '2002' OR year = '2003' THEN CAST(WAR AS FLOAT) ELSE 0 END) > 3
   OR SUM(CASE WHEN year = '2002' OR year = '2003' THEN CAST(WAR AS FLOAT) ELSE 0 END) > 5
ORDER BY combined_WAR DESC;


# Problem 2
.output problem2.csv
SELECT playerID,
       name,
       (CASE WHEN WAR >= 1 THEN 1 ELSE 0 END) AS war_1,
       (CASE WHEN WAR >= 2 THEN 1 ELSE 0 END) AS war_2,
       (CASE WHEN WAR >= 3 THEN 1 ELSE 0 END) AS war_3,
	SUM(CASE WHEN year = '2018' THEN WAR ELSE 0 END) AS total_war
FROM PERF
WHERE Org = 'ATL' AND year = '2018' AND level = 'mlb'
GROUP BY playerID, name;

# Problem 3
.output problem3.csv
SELECT 
    COUNT(DISTINCT GameKey || INNING || PA_OF_INNING) AS total_PA,
    SUM(CASE WHEN STRIKES = '2' AND BALLS = '0' THEN 1 ELSE 0 END) AS count_0_2,
    SUM(CASE WHEN STRIKES = '2' AND BALLS = '1' THEN 1 ELSE 0 END) AS count_1_2,
    SUM(CASE WHEN STRIKES = '2' AND BALLS = '2' THEN 1 ELSE 0 END) AS count_2_2
FROM PITCHBYPITCH

#
WHERE PitcherName = 'Jackson, Luke'
  AND STRIKES = '2'  
  AND IS_STRIKEOUT = '0';  


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
WHERE PitcherName = 'Jackson, Luke'
  AND STRIKES = '2'  
  AND IS_STRIKEOUT = '0';  

  # Open up Google Collab using Python and perform the following code For the Data Wrangling Question Problem #1
  import requests
import pandas as pd

# Step 1: Fetch Data from StatsAPI
url = "https://statsapi.mlb.com/api/v1/stats?stats=season&group=pitching&playerPool=all&season=2018&teamId=144"
response = requests.get(url)

# Check if the request was successful
if response.status_code == 200:
    stats_data = response.json()  # Parse the JSON response
else:
    raise Exception(f"Failed to fetch data from StatsAPI. Status Code: {response.status_code}")

# Step 2: Extract Player Information and Stats
players = []

# Access the nested "splits" data
for split in stats_data.get('stats', [])[0].get('splits', []):
    player_info = split.get('player', {})
    player_stats = split.get('stat', {})
    
    # Gather relevant data for each player
    players.append({
        'playerID': player_info.get('id'),
        'playerName': player_info.get('fullName'),
        'gamesPlayed': player_stats.get('gamesPlayed'),
        'gamesStarted': player_stats.get('gamesStarted'),
        'inningsPitched': player_stats.get('inningsPitched'),
        'strikes': player_stats.get('strikes'),
        'strikeOuts': player_stats.get('strikeOuts'),
        'doubles': player_stats.get('doubles'),
        'triples': player_stats.get('triples'),
        'walks': player_stats.get('baseOnBalls'),
        'hits': player_stats.get('hits'),
        'earnedRuns': player_stats.get('earnedRuns'),
        'homeRuns': player_stats.get('homeRuns'),
        'ERA': player_stats.get('era')
    })

# Step 3: Create a Pandas DataFrame
df = pd.DataFrame(players)
len(df)
print(df)

# Step 4: Save the Data to CSV
csv_file = 'player_pitching_stats_2018.csv'
df.to_csv(csv_file, index=False)

print(f"Data saved to {csv_file}")

# Open SQLite and run the following code for Problem 2 of the Data Wrangling Questions
# Import CSV File into SQlite
.open main
.mode csv
 CREATE TABLE IF NOT EXISTS PLAYER_PITCHING_STATS (
       playerID TEXT,
       playerName TEXT,
       gamesPlayed INTEGER,
       gamesStarted INTEGER,
       inningsPitched REAL,
       strikes INTEGER,
       strikeOuts INTEGER,
       doubles INTEGER,
       triples INTEGER,
       walks INTEGER,
       hits INTEGER,
       earnedRuns INTEGER,
       homeRuns INTEGER,
       ERA REAL
   );
.import "C:/Users/13195/OneDrive/db01/player_pitching_stats_2018 (1).csv" PLAYER_PITCHING_STATS
# Analze Discrepencies in Overall Strikeouts between the 2 tables
.output strikeoutdiscrepencies.csv
SELECT 
    p.PitcherID, 
    p.PitcherName, 
    SUM(CASE WHEN IS_STRIKEOUT = '1' THEN 1 ELSE 0 END) AS Pitching_Strikeouts,
    s.strikeOuts AS StatsAPI_Strikeouts,
    (SUM(CASE WHEN p.IS_STRIKEOUT = '1' THEN 1 ELSE 0 END) - s.strikeOuts) AS Difference
FROM 
    PITCHBYPITCH p
JOIN 
    PLAYER_PITCHING_STATS s ON p.PitcherID = s.playerID
GROUP BY 
    p.PitcherID, p.PitcherName
HAVING 
    Difference != 0;

# Analyze Discrepencise in Earned Runs between the 2 tables
.output earnedrunsdiscrepencies.csv
SELECT 
    p.PitcherID, 
    p.PitcherName, 
    s.earnedRuns AS StatsAPI_EarnedRuns,
    SUM(CASE WHEN p.IS_HIT = '1' THEN 1 ELSE 0 END) AS Hits_Allowed,
    (SUM(CASE WHEN p.IS_HIT = '1' THEN 1 ELSE 0 END) - s.earnedRuns) AS Difference
FROM 
    PITCHBYPITCH p
JOIN 
    PLAYER_PITCHING_STATS s ON p.PitcherID = s.playerID
GROUP BY 
    p.PitcherID, p.PitcherName
HAVING 
    Difference != 0;

# Analyze discrepencies in Inning Pitched between the 2 tables
.output inningspitcheddiscrepencies.csv
SELECT 
    p.PitcherID, 
    p.PitcherName, 
    SUM(CASE WHEN p.INNING IS NOT NULL THEN 1 ELSE 0 END) AS Innings_Pitched_PITCHBYPITCH,
    s.inningsPitched AS StatsAPI_Innings_Pitched,
    (SUM(CASE WHEN p.INNING IS NOT NULL THEN 1 ELSE 0 END) - s.inningsPitched) AS Difference
FROM 
    PITCHBYPITCH p
JOIN 
    PLAYER_PITCHING_STATS s ON p.PitcherID = s.playerID
GROUP BY 
    p.PitcherID, p.PitcherName
HAVING 
    Difference != 0;

# Analyze discrepencies in the Games Played between the 2 tables
.output gamesplayeddiscrepencies.csv
SELECT 
    p.PitcherID, 
    p.PitcherName, 
    COUNT(DISTINCT p.GameKey) AS Games_Played_PITCHBYPITCH,
    s.gamesPlayed AS StatsAPI_Games_Played,
    (COUNT(DISTINCT p.GameKey) - s.gamesPlayed) AS Difference
FROM 
    PITCHBYPITCH p
JOIN 
    PLAYER_PITCHING_STATS s ON p.PitcherID = s.playerID
GROUP BY 
    p.PitcherID, p.PitcherName
HAVING 
    Difference != 0;
# From all 4 of these analyses I was able to conclude that there is inaccurate data present for the pitchers Jonny Venters, Luke Jackson and Shane Carle as they each had discrepencies in each of these analyses showing that there is inaccurate data present. Except for when evaluating Games Played, where Luke Jackson did not have any discrepencies present but Johnny Venters and Shane Carle did. 

# Potential nexts steps I could use in my code to create an audit system would be to first define the metrics that are critical for comparison and determine set thresholds that are acceptable for discrepencies. Next I would add automatic integration into my code so that data from the StatsAPI table website and the PITCHBYPITCH table are automatically updated and uploaded. I would then create queries for each of the critical metrics so that an automatic comparison could be done. I would then set alerts so that if a discrepency is found within the audit system, it can be fixed. And finally I would create a visualization dashboard, such as on Tableau so that I could visually see trends in discrepencies over time, or see where discrepencies may be most likely to occur.
    







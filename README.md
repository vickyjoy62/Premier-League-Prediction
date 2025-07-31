# Premier-League-Prediction
Predicting Premier League Team Performance
#
python
#
api
#
coding
#
datascience
I was working on a project to predict the win probabilities of the Premier League teams based on the current match outcomes. I pulled together match results from the two recent seasons, that is 2023 and 2024.

These are the steps I took in this project.

1. Getting the match data.
I used the https://www.football-data.org/ API to get the matches played in the Premier League in the two seasons. I used these codes to fetch the data from the API.

def fetch_matches(season_year):
url = f"https://api.football-data.org/v4/competitions/PL/matches?season={season_year}"
response = requests.get(url, headers=headers)
data = response.json()

matches = pd.DataFrame(data["matches"])

matches["homeTeam"] = matches["homeTeam"].apply(lambda x: x["name"])
matches["awayTeam"] = matches["awayTeam"].apply(lambda x: x["name"])
matches["score"] = matches["score"].apply(lambda x: x["winner"])  
matches["season"] = season_year

return matches
2. Merging the two seasons.
The data needed to be put into one single dataframe. This is the code I used:

matches_2023 = fetch_matches(2023)
matches_2024 = fetch_matches(2024)
all_matches = pd.concat([matches_2023, matches_2024], ignore_index=True) #merge data frames

3. Calculating wins, draws, and matches played.
I used the groupby operations to count: Home wins, Away wins, Draws and the Matches Played.
This is the code I used:

Wins
home_wins = all_matches[all_matches["score"] == "HOME_TEAM"].groupby("homeTeam").size()
away_wins = all_matches[all_matches["score"] == "AWAY_TEAM"].groupby("awayTeam").size()
total_wins = home_wins.add(away_wins, fill_value=0).astype(int)

Draws
draws = all_matches[all_matches["score"] == "DRAW"]
home_draws = draws.groupby("homeTeam").size()
away_draws = draws.groupby("awayTeam").size()
total_draws = home_draws.add(away_draws, fill_value=0).astype(int)

Matches played
home_matches = all_matches.groupby("homeTeam").size()
away_matches = all_matches.groupby("awayTeam").size()
total_matches = home_matches.add(away_matches, fill_value=0).astype(int)

4. Assigning Points
I calculated each team's total points. In football, 3 points is for a win, 1 point is for a draw.
The code I used was:
total_points = (total_wins * 3).add(total_draws * 1, fill_value=0)

5. Estimating win probabilities
With the total points and total matches, it was possible to find the win probability.
First, I calculated the maximum possible points, that is with 3 points per match.
match_counts = pd.concat([
all_matches["homeTeam"],
all_matches["awayTeam"]
]).value_counts()

max_points = match_counts * 3

Then, I calculated the normalized win by comparing the actual points to the maximum possible points.

probability_win = total_points / (max_points * 20) # Scaling factor for readability
probability_win = probability_win.sort_values(ascending=False)

Final Thoughts
This was a simple project yet a powerful one. Integrating football and data science was interesting, all I needed was an API and python to work it out.

# Instructions
Follow the instructions and get all the user stories below to pass to finish the project.

You start with several files, one of them is `games.csv`. It contains a comma-separated list of all games of the final three rounds of the World Cup tournament since 2014; the titles are at the top. It includes the year of each game, the round of the game, the winner, their opponent, and the number of goals each team scored. You need to do three things for this project:

Part 1: Create the database

Log into the psql interactive terminal with `psql --username=freecodecamp --dbname=postgres` and create your database structure according to the user stories below.

Don't forget to connect to the database after you create it.

Notes:
If you leave your virtual machine, your database may not be saved. You can make a dump of it by entering `pg_dump -cC --inserts -U freecodecamp worldcup > worldcup.sql` in a bash terminal (not the psql one). It will save the commands to rebuild your database in `worldcup.sql`. The file will be located where the command was entered. If it's anything inside the project folder, the file will be saved in the VM. You can rebuild the database by entering `psql -U postgres < worldcup.sql` in a terminal where the .sql file is.

If you are saving your progress on freeCodeCamp.org, after getting all the tests to pass, follow the instructions above to save a dump of your database. Save the worldcup.sql file, as well as the final version of your `insert_data.sh` and `queries.sh` files, in a public repository and submit the URL to it on freeCodeCamp.org.
# Tasks
## Part 1
1. You should create a database named `worldcup`
```sql
CREATE DATABASE worldcup;
```
2. You should connect to your worldcup database and then create `teams` and `games` tables
3. Your teams table should have a `team_id` column that is a type of `SERIAL` and is the primary key, and a `name` column that has to be `UNIQUE`
```sql
CREATE TABLE teams(team_id SERIAL PRIMARY KEY);
ALTER TABLE teams ADD COLUMN name VARCHAR(50) UNIQUE NOT NULL;
```
Output:
```txt
worldcup=> \d teams
                               Table "public.teams"
 Column  |  Type   | Collation | Nullable |                Default                 
---------+---------+-----------+----------+----------------------------------------
 team_id | integer |           | not null | nextval('teams_team_id_seq'::regclass)
Indexes:
    "teams_pkey" PRIMARY KEY, btree (team_id)
```
4. Your `games` table should have a `game_id` column that is a type of `SERIAL` and is the primary key, a year column of type `INT`, and a round column of type `VARCHAR`
```sql
CREATE TABLE games(game_id SERIAL PRIMARY KEY);
ALTER TABLE games ADD COLUMN round VARCHAR(50) NOT NULL;
ALTER TABLE games ADD COLUMN year INT NOT NULL;
```
Output:
```txt
Table "public.games"
 Column  |         Type          | Collation | Nullable |                Default                 
---------+-----------------------+-----------+----------+----------------------------------------
 game_id | integer               |           | not null | nextval('games_game_id_seq'::regclass)
 round   | character varying(50) |           | not null | 
 year    | integer               |           | not null | 
Indexes:
    "games_pkey" PRIMARY KEY, btree (game_id
```
```txt
Table "public.teams"
 Column  |         Type          | Collation | Nullable |                Default                 
---------+-----------------------+-----------+----------+----------------------------------------
 team_id | integer               |           | not null | nextval('teams_team_id_seq'::regclass)
 name    | character varying(50) |           | not null | 
Indexes:
    "teams_pkey" PRIMARY KEY, btree (team_id)
    "teams_name_key" UNIQUE CONSTRAINT, btree (name)
```
5. Your `games` table should have `winner_id` and `opponent_id` foreign key columns that each reference `team_id` from the `teams` table
```sql
ALTER TABLE games ADD COLUMN winner_id INT REFERENCES teams(team_id);
ALTER TABLE games ADD COLUMN opponent_id INT REFERENCES teams(team_id);
```
6. Your `games` table should have `winner_goals` and `opponent_goals` columns that are type `INT`
```sql
ALTER TABLE games ADD COLUMN winner_goals INT NOT NULL;
ALTER TABLE games ADD COLUMN opponent_goals INT NOT NULL;
```
Output:
```txt
Table "public.games"
     Column     |         Type          | Collation | Nullable |                Default                 
----------------+-----------------------+-----------+----------+----------------------------------------
 game_id        | integer               |           | not null | nextval('games_game_id_seq'::regclass)
 round          | character varying(50) |           | not null | 
 year           | integer               |           | not null | 
 winner_id      | integer               |           |          | 
 opponent_id    | integer               |           |          | 
 winner_goals   | integer               |           | not null | 
 opponent_goals | integer               |           | not null | 
Indexes:
    "games_pkey" PRIMARY KEY, btree (game_id)
Foreign-key constraints:
    "games_opponent_id_fkey" FOREIGN KEY (opponent_id) REFERENCES teams(team_id)
    "games_winner_id_fkey" FOREIGN KEY (winner_id) REFERENCES teams(team_id)
```
7. All of your columns should have the `NOT NULL` constraint
```sql
ALTER TABLE games ALTER COLUMN winner_id SET NOT NULL;
ALTER TABLE games ALTER COLUMN opponent_id SET NOT NULL;
```
Output:
```txt
Table "public.games"
     Column     |         Type          | Collation | Nullable |                Default                 
----------------+-----------------------+-----------+----------+----------------------------------------
 game_id        | integer               |           | not null | nextval('games_game_id_seq'::regclass)
 round          | character varying(50) |           | not null | 
 year           | integer               |           | not null | 
 winner_id      | integer               |           | not null | 
 opponent_id    | integer               |           | not null | 
 winner_goals   | integer               |           | not null | 
 opponent_goals | integer               |           | not null | 
Indexes:
    "games_pkey" PRIMARY KEY, btree (game_id)
Foreign-key constraints:
    "games_opponent_id_fkey" FOREIGN KEY (opponent_id) REFERENCES teams(team_id)
    "games_winner_id_fkey" FOREIGN KEY (winner_id) REFERENCES teams(team_id)
```
8. Your two script (.sh) files should have executable permissions. Other tests involving these two files will fail until permissions are correct. When these permissions are enabled, the tests will take significantly longer to run
```sh
chmod +x insert_data.sh
chmod +x queries.sh
```
## Part 2
Part 2: Insert the data

Complete the `insert_data.sh` script to correctly insert all the data from games.csv into the database. The file is started for you. Do not modify any of the code you start with. Using the PSQL variable defined, you can make database queries like this: `$($PSQL "<query_here>")`. The tests have a 20 second limit, so try to make your script efficient. The less you have to query the database, the faster it will be. You can empty the rows in the tables of your database with `TRUNCATE TABLE games, teams`;

9. When you run your `insert_data.sh` script, it should add each unique team to the teams table. There should be 24 rows
10. When you run your `insert_data.sh` script, it should insert a row for each line in the games.csv file (other than the top line of the file). There should be 32 rows. Each row should have every column filled in with the appropriate info. Make sure to add the correct ID's from the teams table (you cannot hard-code the values)
```sh
#! /bin/bash

if [[ $1 == "test" ]]
then
  PSQL="psql --username=postgres --dbname=worldcuptest -t --no-align -c"
else
  PSQL="psql --username=freecodecamp --dbname=worldcup -t --no-align -c"
fi
# Do not change code above this line. Use the PSQL variable above to query your database.
echo $($PSQL "TRUNCATE games, teams")
cat games.csv | while IFS="," read YEAR ROUND WINNER OPPONENT WINNER_GOALS OPPONENT_GOALS
do
  if [[ $YEAR != "year" ]]
  then

    # get winner_id
    WINNER_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$WINNER'")

    # if not found
    if [[ -z $WINNER_ID ]]
    then
      # insert winner
      INSERT_WINNER_TEAM=$($PSQL "INSERT INTO teams(name) VALUES('$WINNER')")

      # get new winner_id
      WINNER_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$WINNER'")
    fi

    # get opponent_id
    OPPONENT_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$OPPONENT'")

    # if not found
    if [[ -z $OPPONENT_ID ]]
    then
      # insert opponent
      INSERT_OPPONENT_TEAM=$($PSQL "INSERT INTO teams(name) VALUES('$OPPONENT')")

      # get new opponent_id
      OPPONENT_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$OPPONENT'")
    fi

    INSERT_MAJORS_COURSES_RESULT=$($PSQL "INSERT INTO games(year, round, winner_id, opponent_id, winner_goals, opponent_goals) VALUES($YEAR, '$ROUND', $WINNER_ID, $OPPONENT_ID, $WINNER_GOALS, $OPPONENT_GOALS)")

  fi

done
```
When you type `SELECT * FROM teams` and `SELECT * FROM games`, the outputs are:
```txt
 team_id |     name      
---------+---------------
       1 | France
       2 | Croatia
       3 | Belgium
       4 | England
       5 | Russia
       6 | Sweden
       7 | Brazil
       8 | Uruguay
       9 | Colombia
      10 | Switzerland
      11 | Japan
      12 | Mexico
      13 | Denmark
      14 | Spain
      15 | Portugal
      16 | Argentina
      17 | Germany
      18 | Netherlands
      19 | Costa Rica
      20 | Chile
      21 | Nigeria
      22 | Algeria
      23 | Greece
      24 | United States
(24 rows)
```
```txt
 game_id |     round     | year | winner_id | opponent_id | winner_goals | opponent_goals 
---------+---------------+------+-----------+-------------+--------------+----------------
       1 | Final         | 2018 |         1 |           2 |            4 |              2
       2 | Third Place   | 2018 |         3 |           4 |            2 |              0
       3 | Semi-Final    | 2018 |         2 |           4 |            2 |              1
       4 | Semi-Final    | 2018 |         1 |           3 |            1 |              0
       5 | Quarter-Final | 2018 |         2 |           5 |            3 |              2
       6 | Quarter-Final | 2018 |         4 |           6 |            2 |              0
       7 | Quarter-Final | 2018 |         3 |           7 |            2 |              1
       8 | Quarter-Final | 2018 |         1 |           8 |            2 |              0
       9 | Eighth-Final  | 2018 |         4 |           9 |            2 |              1
      10 | Eighth-Final  | 2018 |         6 |          10 |            1 |              0
      11 | Eighth-Final  | 2018 |         3 |          11 |            3 |              2
      12 | Eighth-Final  | 2018 |         7 |          12 |            2 |              0
      13 | Eighth-Final  | 2018 |         2 |          13 |            2 |              1
      14 | Eighth-Final  | 2018 |         5 |          14 |            2 |              1
      15 | Eighth-Final  | 2018 |         8 |          15 |            2 |              1
      16 | Eighth-Final  | 2018 |         1 |          16 |            4 |              3
      17 | Final         | 2014 |        17 |          16 |            1 |              0
      18 | Third Place   | 2014 |        18 |           7 |            3 |              0
      19 | Semi-Final    | 2014 |        16 |          18 |            1 |              0
      20 | Semi-Final    | 2014 |        17 |           7 |            7 |              1
      21 | Quarter-Final | 2014 |        18 |          19 |            1 |              0
      22 | Quarter-Final | 2014 |        16 |           3 |            1 |              0
      23 | Quarter-Final | 2014 |         7 |           9 |            2 |              1
      24 | Quarter-Final | 2014 |        17 |           1 |            1 |              0
      25 | Eighth-Final  | 2014 |         7 |          20 |            2 |              1
      26 | Eighth-Final  | 2014 |         9 |           8 |            2 |              0
      27 | Eighth-Final  | 2014 |         1 |          21 |            2 |              0
      28 | Eighth-Final  | 2014 |        17 |          22 |            2 |              1
      29 | Eighth-Final  | 2014 |        18 |          12 |            2 |              1
      30 | Eighth-Final  | 2014 |        19 |          23 |            2 |              1
      31 | Eighth-Final  | 2014 |        16 |          10 |            1 |              0
      32 | Eighth-Final  | 2014 |         3 |          24 |            2 |              1
(32 rows)
```
See full source code [here](insert_data.sh)
## Part 3
Part 3: Query the database

Complete the empty `echo` commands in the `queries.sh` file to produce output that matches the `expected_output.txt` file. The file has some starter code, and the first query is completed for you. Use the PSQL variable defined to complete rest of the queries. Note that you need to have your database filled with the correct data from the script to get the correct results from your queries. Hint: Test your queries in the psql prompt first and then add them to the script file.
```sh
#! /bin/bash

PSQL="psql --username=freecodecamp --dbname=worldcup --no-align --tuples-only -c"

# Do not change code above this line. Use the PSQL variable above to query your database.

echo -e "\nTotal number of goals in all games from winning teams:"
echo "$($PSQL "SELECT SUM(winner_goals) FROM games")"

echo -e "\nTotal number of goals in all games from both teams combined:"
echo "$($PSQL "SELECT SUM(winner_goals+opponent_goals) FROM games")"

echo -e "\nAverage number of goals in all games from the winning teams:"
echo "$($PSQL "SELECT AVG(winner_goals) FROM games")"

echo -e "\nAverage number of goals in all games from the winning teams rounded to two decimal places:"
echo "$($PSQL "SELECT ROUND(AVG(winner_goals),2) FROM games")"

echo -e "\nAverage number of goals in all games from both teams:"
echo "$($PSQL "SELECT AVG(winner_goals+opponent_goals) FROM games")"

echo -e "\nMost goals scored in a single game by one team:"
echo "$($PSQL "SELECT MAX(winner_goals) FROM games")"

echo -e "\nNumber of games where the winning team scored more than two goals:"
echo "$($PSQL "SELECT count(*) FROM games where winner_goals>2")"

echo -e "\nWinner of the 2018 tournament team name:"
echo "$($PSQL "select name from teams full join games on teams.team_id = games.winner_id where year=2018 and round='Final'")"

echo -e "\nList of teams who played in the 2014 'Eighth-Final' round:"
echo "$($PSQL "select name from teams full join games on ( teams.team_id = games.winner_id or teams.team_id = games.opponent_id) where year=2014 and round='Eighth-Final' order by name")"

echo -e "\nList of unique winning team names in the whole data set:"
echo "$($PSQL "select distinct(name) from teams inner join games on teams.team_id = games.winner_id order by name")"

echo -e "\nYear and team name of all the champions:"
echo "$($PSQL "select year,name from teams inner join games on teams.team_id = games.winner_id where round='Final' order by year")"

echo -e "\nList of teams that start with 'Co':"
echo "$($PSQL "select name from teams where name like 'Co%'")"
```
Output:
```txt

Total number of goals in all games from winning teams:
68

Total number of goals in all games from both teams combined:
90

Average number of goals in all games from the winning teams:
2.1250000000000000

Average number of goals in all games from the winning teams rounded to two decimal places:
2.13

Average number of goals in all games from both teams:
2.8125000000000000

Most goals scored in a single game by one team:
7

Number of games where the winning team scored more than two goals:
6

Winner of the 2018 tournament team name:
France

List of teams who played in the 2014 'Eighth-Final' round:
Algeria
Argentina
Belgium
Brazil
Chile
Colombia
Costa Rica
France
Germany
Greece
Mexico
Netherlands
Nigeria
Switzerland
United States
Uruguay

List of unique winning team names in the whole data set:
Argentina
Belgium
Brazil
Colombia
Costa Rica
Croatia
England
France
Germany
Netherlands
Russia
Sweden
Uruguay

Year and team name of all the champions:
2014|Germany
2018|France

List of teams that start with 'Co':
Colombia
Costa Rica
```

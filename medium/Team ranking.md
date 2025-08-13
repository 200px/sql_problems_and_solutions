## Team ranking
### Introduction
There's a team competition going on and your task is to fetch the current team ranking. 

As teams can have different amounts of members, it would be unfair if huge teams had advantages over smaller teams.

Therefore we are only considering the scores of the best 5 members per team. If a team consists of less than 5 members, all of its members' scores are summed up.

### Task
Fetch the 10 best teams with their aggregated team scores (only considering the top 5 scores per team) and also the top 5 team members each with their respective score (ordered by highest score first) within the team. 

See the example for details.

### Requirements
In order to get a deterministic result, the following rules apply

### Result Structure
team_rank (int): the rank, starting with 1

team_name (text): name of the team

team_score (int): the sum of scores of the best 5 members of this team

top_members (text): comma-separated list of the best 5 members (name + score)

### Teams
the 10 best teams must be ordered by their score (highest to lowest)

if two or more teams end up with the same team score, both get the same rank assigned, but the one with the lower team ID is listed first

only up to 10 teams must be listed: if there are multiple teams sharing the 10th place, the one with the lower team ID must be preferred
### Team Members
within the teams, the members must be ordered by their score (highest to lowest)

if two or more members have the same score, the one with the lower ID is listed first

only up to 5 members per team must be listed; if there are multiple members sharing the 5th place, the one with the lower ID must be preferred

### Example

**teams**

| id | name                |
|----|---------------------|
| 1  | Adorable Axolotl    |
| 2  | Brave Buffalo       |
| 3  | Colorful Chipmunk   |
| 4  | Delightful Dolphin  |

**team_members**

| id | team_id | name     | score |
|----|---------|----------|-------|
| 1  | 1       | Alice    | 14    |
| 2  | 1       | Bill     | 12    |
| 3  | 1       | Chuck    | 7     |
| 4  | 1       | Daryl    | 13    |
| 5  | 1       | Emilia   | 18    |
| 6  | 1       | Frank    | 5     |
| 7  | 1       | Gracie   | 16    |
| 8  | 2       | Hanna    | 33    |
| 9  | 2       | Isaac    | 41    |
| 10 | 3       | Jonathan | 36    |
| 11 | 4       | Keyla    | 21    |
| 12 | 4       | Lee      | 18    |
| 13 | 4       | Marcus   | 26    |

**Expected Output**
| team_rank | team_name          | team_score | top_members                                                 |
|:---------:|:-------------------|:----------:|:------------------------------------------------------------|
|     1     | Brave Buffalo      |     74     | Isaac (41), Hanna (33)                                      |
|     2     | Adorable Axolotl   |     73     | Emilia (18), Gracie (16), Alice (14), Daryl (13), Bill (12) |
|     3     | Delightful Dolphin |     65     | Marcus (26), Keyla (21), Lee (18)                             |
|     4     | Colorful Chipmunk  |     36     | Jonathan (36)                                               |


## My solution to the problem
~~~~sql
with cte as 
(SELECT name, team_id, score,
name || ' (' || score || ')' AS name_with_score, 
ROW_NUMBER() OVER (partition by team_id order by score DESC, id)  as place_in_team
FROM team_members)


SELECT 
RANK() OVER (order by SUM(score) DESC) as team_rank,
SUM(score) as team_score,
teams.name as team_name, 
STRING_AGG(name_with_score, ', ') as top_members 
FROM cte
JOIN teams ON teams.id = cte.team_id
WHERE place_in_team <= 5
GROUP BY teams.name, team_id
ORDER BY team_rank, team_id
LIMIT 10
~~~~
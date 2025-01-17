---
title: "Predict win/lose of each game of March Madness for year 2018"
author: Team Scorpion - June Kim (jkim654), Hoyin Lau (hlau4), Xiaomei Sun (xsun56),
  Taiga Hasegawa (taigah2), Leo Franco Soto (francst2)
date: "5/5/2019"
output: word_document
---

# Introduction 

The NCAA Division I Men's Basketball Tournament, also known as NCAA March Madness happens every year in spring. It features 68 college basketball teams from Division I level of the National Collegiate Atheletic Association (NCAA) to determine the national championship. March Madness was first created in 1939 by the National Association of Basketball Coaches and was pitched by Harold Olsen. And currently millions of people in America fill out a bracket to correctly predict the outcome of the entire event.

The format of the tournament are in rounds in the following order[1]:

- The First Four
- The First Round (the Round of 64)
- The Second Round (the Round of 32)
- The Regional Semi-finals (participating teams are known popularly as the "Sweet Sixteen")
- The Regional Finals (participating teams are known commonly as the "Elite Eight")
- The National Semi-finals (participating teams are referred to officially as the "Final Four")
- The National Championship

The motivation behind this project is to correctly predict the outcome of a sports game. The ability to accurately predict the win/loss could help sports betting significantly. We will utilize the basketball game data, like the number of 2 pointers/3 pointers in a given game, to draw out useful information and provide insight on the outcome of each game, which is invaluable to sports bettors and viewers who just want to have a fun bet with friends. The data is from Kaggle Machine Learning Competition hosted by Google Cloud and NCAA 2018. It is a historical data collecting from year 1985 to 2018 (i.e. season 2017-2018, since this year's season is 2018-2019)[2].

There are other people who use the same dataset to do different analyses. The most common topics are the overall rating of teams and players, predicting future best team, how different coaches will have impact on the performance of the teams. These are the topics most people have interest with and will spend a lot of time to discuss about. There are lots of very interesting analyses, and they are totally different from the others. They are trying to predict the salary of different players based on their performance in game, how long are they staying, and how well they do from time to time. There is also a prediction of the salary of different coaches based on how well their coaching team performed. Similarly, there are also other people analyze the win and lose rate, but the way we analyze is different from other people. We are using the overall past data, such as scores from the pasts, location, number of three points and free throws and many other variables to predict the win rate, while other people only used the current data or only few variables to predict the win rate. In this project, the goal is was to find the most accurate model to find the winning predictions for NCAA March Madness in year 2018. 

# Data Exploration

The original data from Kaggle Machine Learning Competitions had 65 variables with 204,861 observations. For this project, we have cleaned the dataset leaving us with 48 variables and 204,861 observations. The links to the cleaned datasets are below:

[Training Data](https://drive.google.com/open?id=1henbg-CbVdXcr8jnsLH7JRA0AuWriPoD)
[Testing data](https://drive.google.com/file/d/1p6nr-kGy-orNEP8fEn0YjNgdI9Spo9aq/view?usp=sharing)

The training dataset has 204,727 observations and the testing dataset has 134 observations.

The dataset contains information about the regular season before the March Madness and studies on the performance on regular seasons for the teams presented directly influences whether or not they make it to the March Madness itself. Therefore, we have conducted seasonal analysis and overall performance of each team before jumping into NCAA Tour results. 

## Season Analysis 

How teams perform during regular seasons affect their chances of getting into the March Madness. In this part, we have analyzed the seasonal performance of the teams that have historically made it into March Madness. First, we explored the relationship between the location of the game with the percentage of wins. The goal here was to find if percentage of wins are associated with the location. 

```{r,echo=FALSE}
  RegularSeasonDetailedResults = read.csv("data/PrelimData2018/RegularSeasonDetailedResults_Prelim2018.csv")
  Teams = read.csv("data/DataFiles/Teams.csv")
```

```{r,echo=FALSE,error=FALSE}
  
  library(plyr)
  library(ggplot2)
  
  # calculate frequency
  loc_freq = count(RegularSeasonDetailedResults$WLoc)
  
  # calculate percentage and round to 2sf
  loc_freq['percentage'] = 100/sum(loc_freq$freq)*loc_freq$freq
  loc_freq['percentage_r'] <- signif(loc_freq$percentage, digits = 3)
  
  # calculate center of each segment for label
  loc_freq['pos'] <- cumsum(loc_freq$percentage) - loc_freq$percentage/2
  # set levels for segments so can order to match labels
  loc_freq$x <- factor(loc_freq$x, levels = rev(loc_freq$x))
  
  # pie chart
  ggplot(loc_freq, aes(x="", y=percentage_r, fill = x))+
    geom_bar(stat="identity")+coord_polar("y")+
    ggtitle("% of games won at each location")+
    geom_text(aes(y=pos, label=percentage_r))+theme(
      axis.title.x = element_blank(),
      axis.title.y = element_blank(),
      axis.ticks = element_blank(),
      axis.text = element_blank(),
      panel.border = element_blank(),
      panel.grid = element_blank()
    )
```
From looking at the graph, it is possible to witness 59.5% of the games won were at home while 30.6% were games won away. Neutral locations seem to be the worst options for teams to win, leaving 9.9% of the games won at this location. From this, we can see that locations may have a big impact in the chance of winning. 

## Team Analysis

Teams can be evaluated in terms of (1) goals they score in average or (2) win percentage across seasons. First, we considered the win percentage across the regular season. The questions that could be answered from this analysis are: who are the top 34 teams that frequently wins the game in the regular seasons? Are they ones who make it to the March Madness most frequently? 

```{r, echo = FALSE}
# win freq
team_Wfreq = count(RegularSeasonDetailedResults$WTeamID)
team_Wfreq$x = as.factor(team_Wfreq$x)

# lose freq
team_Lfreq = count(RegularSeasonDetailedResults$LTeamID)
team_Lfreq$x  = as.factor(team_Lfreq$x)

# rename common name to merge
colnames(team_Wfreq)[1] = "TeamID"
colnames(team_Lfreq)[1] = "TeamID"
team_Wfreq = merge(team_Wfreq, Teams, by = "TeamID")
win_freq = merge(team_Wfreq,team_Lfreq, by = "TeamID")
final_frequency = win_freq[-4:-5]
colnames(final_frequency)[2] = "Win"
colnames(final_frequency)[4] = "Lose"

# calculate win percentage
final_frequency["percentage"] = (100 / (final_frequency$Win + final_frequency$Lose)) * final_frequency$Win
final_frequency$TeamID <- as.factor(final_frequency$TeamID)
final_frequency$TeamID <- factor(final_frequency$TeamID, levels = final_frequency$TeamID[order(final_frequency$percentage)])

## Question: What are top 34 teams? 
top.teams = head(final_frequency[order(-final_frequency$percentage),], n=34)

## Question: What are worst 34 teams?
bottom.teams = head(final_frequency[order(final_frequency$percentage),], n=34)

# plot
 ggplot(top.teams, aes(x=TeamName, y=percentage)) +
  ylim(0, 100) +
  ggtitle("Top 34 teams % of games won") +
  geom_bar(stat='identity') +
  xlab("Team Name") + theme(
    axis.text.x = element_text(angle=90, hjust=1)
  )
 
 # plot
 ggplot(bottom.teams, aes(x=TeamName, y=percentage)) +
  ylim(0, 100) +
  ggtitle("Worst 34 teams % of games won") +
  geom_bar(stat='identity') +
  xlab("Team Name") + theme(
    axis.text.x = element_text(angle=90, hjust=1)
  )
```

From this exploration, it was possible to see that the top 34 teams with the highest % of games won are one of the most frequent teams playing at NCAA March Madness. For instance, Vilanova won in 2018 and it is one of the top 34 teams. Duke, who is on the top 34 teams, is also very strong team, always making it to the Elite Eight most of the seasons. Moreover, when we look at the worst 34 teams, the winning % of games are in average 25-30%. 

```{r, echo = FALSE}
library(knitr)

Wscores = as.data.frame(RegularSeasonDetailedResults$WTeamID)
colnames(Wscores)[1] = "TeamID"
Wscores["Score"] = RegularSeasonDetailedResults$WScore
Lscores = as.data.frame(RegularSeasonDetailedResults$LTeamID)
colnames(Lscores)[1] = "TeamID"
Lscores["Score"] = RegularSeasonDetailedResults$LScore
AllScores = cbind(Wscores,Lscores)
# calculate team with highest score
AggScores = aggregate(Score ~ TeamID, AllScores,FUN="mean")
AggScores = merge(AggScores, Teams, by = "TeamID")
AggScores$TeamID = as.factor(AggScores$TeamID)
AggScores$TeamID = factor(AggScores$TeamID, levels = AggScores$TeamID[order(AggScores$Score)])
AggScores = AggScores[,-4:-5]

# Top 34 scoring teams
top.score.teams = head(AggScores[order(-AggScores$Score), ], n=34)

# Worst 34 scoring teams
worst.score.teams = head(AggScores[order(AggScores$Score), ], n=34)

# plot
 ggplot(top.score.teams, aes(x=TeamName, y=Score)) +
  ylim(0, 100) +
  ggtitle("Top 34 teams Score per game") +
  geom_bar(stat='identity') +
  xlab("Team Name") + theme(
    axis.text.x = element_text(angle=90, hjust=1)
  )
 
 # plot
 ggplot(worst.score.teams, aes(x=TeamName, y=Score)) +
  ylim(0, 100) +
  ggtitle("Worst 34 teams Score per game") +
  geom_bar(stat='identity') +
  xlab("Team Name") + theme(
    axis.text.x = element_text(angle=90, hjust=1)
  )

# Teams who were in both top 34 scoring teams & top 34 % win teams
kable(top.teams$TeamName[top.teams$TeamName %in% top.score.teams$TeamName], col.names = "Team Name")
```
From observing the scores per game of the top 34 highest scoring team per game and those in the bottom 34, we can actually see there are some teams we observed in top % winning teams and bottom % winning teams. It is possible that those who are both in the list of top scoring and % winning would be also in the NCAA March Madness would be the ones making it to the *Final Four*. Just for a side note, the top five frequent winners of NCAA March Madness were: Kentucky, Kansas, North Carolina, Duke and Temple. Since 2015, Gonzaga has been performing well, making it to Elite Eight most of the seasons. 

Another question we had was to see if points scored are correlated to the game result. To answer this question, we compared the game result relative to the total scores. 

```{r, echo = FALSE}
Wscor <- as.data.frame(RegularSeasonDetailedResults$WTeamID)
colnames(Wscor)[1] <- "TeamID"
Wscor['Score'] <- RegularSeasonDetailedResults$WScore
Wscor['Result'] <- 'Win'

Lscor <- as.data.frame(RegularSeasonDetailedResults$LTeamID)
colnames(Lscor)[1] <- "TeamID"
Lscor['Score'] <- RegularSeasonDetailedResults$LScore
Lscor['Result'] <- 'Lose'

AllScores2 = rbind(Lscor,Wscor)

# Game result relative to total scores
ggplot(AllScores2, aes(x=Score,y=Result,fill=Result,colour=Result)) +
  xlim(0,150) + ggtitle("Result of the match to points scored") +
  geom_point(stat="identity")+
  xlab("Point Scored in Game")+
  ylab("Result")
```

From this graph, we can see that winning matches have higher average points scored in the game. However, there are some outliers where it was a lost game but point scored is very high. 

```{r,echo=FALSE}
# Calculate win probability based on scores
Wscores_freq <- as.data.frame(count(Wscor$Score))
Lscores_freq <- as.data.frame(count(Lscor$Score))
Scores_prob <- merge(Wscores_freq, Lscores_freq, by=c("x"="x"), all=TRUE)
colnames(Scores_prob)<- c("score","win","lose")

# function for calculating probability
probability <- function(x){
     x$win / (x$win + x$lose)
}
Scores_prob['probability'] <- probability(Scores_prob)

# What is the relationship in graph?
plot(Scores_prob$score,Scores_prob$probability,pch=19,xlab="Score",ylab="Probability of winning",main="Chances of winning relationship with Scores",col="darkblue")
```

Now, digging into the relationship between points scored vs. probability of winning, we can see that there is a point in which teams can increase the chance of winning. The pinnacle would be between 80-100. After that, the chance of winning slowly drops. So from this, we can see that "high points scored" doesn't necessarily guarantee "winning". 

## Winning Probability by Basketball Statistics

Also, by exploring different basketball statistics that could be engineered further down the analysis, we can figure out which statistics affect the result of the match. Game statistics are those that may contribute to the probability of winning. Points scores in game (pts), number of assists (ast), number of steals (stl) and number of blocks (blk) are some of the examples of "game statistics" in this report. 

For the purpose of exploration, we looked at 14 variables: 
            * pts: points scored in game
            * fgm: field goals made
            * fgm3: 3-point field goals made
            * fga: field goals attempted
            * fga3: 3-point field goals attempted
            * ftm: free throws made
            * fta: free throws attempted
            * ast: number of assists
            * or: number of offensive rebounds
            * dr: number of defensive rebounds
            * to: number of turn-over
            * stl: number of steals
            * blk: number of blocks
            * pf: number of personal fouls

```{r,warning=FALSE,message=FALSE,echo=FALSE}
library(gridExtra)

# win probability based on fgm
wgfm = as.data.frame(count(RegularSeasonDetailedResults$WFGM))
lgfm = as.data.frame(count(RegularSeasonDetailedResults$LFGM))
fgm_prob = merge(wgfm,lgfm,by=c("x"="x"),all=T)
names(fgm_prob) = c("fgm","win","lose")
fgm_prob["prob"] = probability(fgm_prob)

# win probability based on fgm3
wgfm3 = as.data.frame(count(RegularSeasonDetailedResults$WFGM3))
lgfm3 = as.data.frame(count(RegularSeasonDetailedResults$LFGM3))
fgm3_prob = merge(wgfm3,lgfm3,by=c("x"="x"),all=T)
names(fgm3_prob) = c("fgm3","win","lose")
fgm3_prob["prob"] = probability(fgm3_prob)

# win probability based on fga 
wfga = as.data.frame(count(RegularSeasonDetailedResults$WFGA))
lfga = as.data.frame(count(RegularSeasonDetailedResults$LFGA))
fga_prob = merge(wfga,lfga,by=c("x"="x"),all=T)
names(fga_prob) = c("fga","win","lose")
fga_prob["prob"] = probability(fga_prob)

# win probability based on fga3
wfga3 = as.data.frame(count(RegularSeasonDetailedResults$WFGA3))
lfga3 = as.data.frame(count(RegularSeasonDetailedResults$LFGA3))
fga3_prob = merge(wfga,lfga,by=c("x"="x"),all=T)
names(fga3_prob) = c("fga3","win","lose")
fga3_prob["prob"] = probability(fga3_prob)

# win probability based on ftm
wftm = as.data.frame(count(RegularSeasonDetailedResults$WFTM))
lftm = as.data.frame(count(RegularSeasonDetailedResults$LFTM))
ftm_prob = merge(wftm,lftm,by=c("x"="x"),all=T)
names(ftm_prob) = c("ftm","win","lose")
ftm_prob["prob"] = probability(ftm_prob)

# win probability based on fta
wfta = as.data.frame(count(RegularSeasonDetailedResults$WFTA))
lfta = as.data.frame(count(RegularSeasonDetailedResults$LFTA))
fta_prob = merge(wfta,lfta,by=c("x"="x"),all=T)
names(fta_prob) = c("fta","win","lose")
fta_prob["prob"] = probability(fta_prob)

# win probability based on ast
wast = as.data.frame(count(RegularSeasonDetailedResults$WAst))
last = as.data.frame(count(RegularSeasonDetailedResults$LAst))
ast_prob = merge(wast,last,by=c("x"="x"),all=T)
names(ast_prob) = c("ast","win","lose")
ast_prob["prob"] = probability(ast_prob)

# win probability based on or
wor = as.data.frame(count(RegularSeasonDetailedResults$WOR))
lor  = as.data.frame(count(RegularSeasonDetailedResults$LOR))
or_prob = merge(wor,lor,by=c("x"="x"),all=T)
names(or_prob) = c("or","win","lose")
or_prob["prob"] = probability(or_prob)

# win probability based on dr
wdr = as.data.frame(count(RegularSeasonDetailedResults$WDR))
ldr = as.data.frame(count(RegularSeasonDetailedResults$LDR))
dr_prob = merge(wdr,ldr,by=c("x"="x"),all=T)
names(dr_prob) = c("dr","win","lose")
dr_prob["prob"] = probability(dr_prob)

# win probability based on to
wto = as.data.frame(count(RegularSeasonDetailedResults$WTO))
lto = as.data.frame(count(RegularSeasonDetailedResults$LTO))
to_prob = merge(wto,lto,by=c("x"="x"),all=T)
names(to_prob) = c("to","win","lose")
to_prob["prob"] = probability(to_prob)

# win probability based on stl
wstl = as.data.frame(count(RegularSeasonDetailedResults$WStl))
lstl = as.data.frame(count(RegularSeasonDetailedResults$LStl))
stl_prob = merge(wstl,lstl,by=c("x"="x"),all=T)
names(stl_prob) = c("stl","win","lose")
stl_prob["prob"] = probability(stl_prob)

# win probability based on blk
wblk = as.data.frame(count(RegularSeasonDetailedResults$WBlk))
lblk = as.data.frame(count(RegularSeasonDetailedResults$LBlk))
blk_prob = merge(wblk,lblk,by=c("x"="x"),all=T)
names(blk_prob) = c("blk","win","lose")
blk_prob["prob"] = probability(blk_prob)

# win probability based on pf
wpf = as.data.frame(count(RegularSeasonDetailedResults$WPF))
lpf = as.data.frame(count(RegularSeasonDetailedResults$LPF))
pf_prob = merge(wpf,lpf,by=c("x"="x"),all=T)
names(pf_prob) = c("pf","win","lose")
pf_prob["prob"] = probability(pf_prob)
```

```{r,warning=FALSE,message=FALSE,echo=FALSE}
library(gridExtra)

#---graph---#
p1 <- ggplot(Scores_prob, aes(x=score, y=probability)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("Points Scored in Game") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p2 <- ggplot(fgm_prob, aes(x=fgm, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("fgm") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p3 <- ggplot(fga_prob, aes(x=fga, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("fga") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p4 <- ggplot(fgm3_prob, aes(x=fgm3, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("fgm3") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p5 <- ggplot(fga3_prob, aes(x=fga3, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("fga3") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p6 <- ggplot(ftm_prob, aes(x=ftm, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("ftm") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p7 <- ggplot(fta_prob, aes(x=fta, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("fta") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p8 <- ggplot(or_prob, aes(x=or, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("or") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p9 <- ggplot(dr_prob, aes(x=dr, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("dr") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p10 <- ggplot(ast_prob, aes(x=ast, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("ast") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p11 <- ggplot(to_prob, aes(x=to, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("to") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p12 <- ggplot(stl_prob, aes(x=stl, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("stl") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p13 <- ggplot(blk_prob, aes(x=blk, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("blk") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)
p14 <- ggplot(pf_prob, aes(x=pf, y=prob)) +
  geom_point(stat='identity') +
  geom_smooth() +
  ylim(0, 1) +
  xlab("pf") + 
  ylab('Win Probability') +
  theme(axis.title.x = element_text(size=11), axis.title.y = element_text(size=11)) +
  theme(aspect.ratio=1)

grid.arrange(p1, p2, p3, p4, p5,p6)
grid.arrange(p7,p8,p9,p10,p11,p12)
grid.arrange(p13,p14)
```

Chance of winning increases as more free throws are attempted (fta), more defensive rebounds (dr) are made, more assists (ast) are made, blocks (blk) are made in the game, steals (stl) are made, more field goals are made (fgm, fgm3) and free throws are made (ftm). However, we can see that *attempts* don't help in the games. It is the accuracy of making the goals that matters as any *attempts* give the opponent the chance to rebound (possibly offensive one). Therefore, those teams who have the most accurate scorer in the team would most likely have higher chances of winning. Moreover, since assists and rebounds take enormous team work, we can infer that top teams that actually make it to the March Madness would have very constructive team. 

# Data Modeling 

Our goal is to get the most accurate predictions of NCAA tournament results in year 2018. To achieve this goal, we tried several machine learning methods, such as logistic regression, random forest, LDA and so on. Data modeling was divided into 3 parts: "Data Cleaning", "Feature Engineering", and "Statistical Modeling".  

## Data Cleaning

As we had already discussed, this project used many datasets and it was necessary to merge them and extract important features. First let's look through the dataset. The followings are the files we used. The data regarding the matchup before NCAA tournaments in 2018 was used as train data and the one of NCAA tournaments in 2018 was used as test data.  
  
("test_train.png")

**(Train data)**

  - RegularSeasonCompactResults_Prelim2018.csv: Game-by-game results for many *seasons* of historical data, starting with the 1985 season. For each season, the file includes   all games played from daynum 0 through 132. It is important to realize that the "Regular Season" games are simply defined to be all games played on DayNum=132 or earlier (DayNum=132 is Selection Sunday) 
  
  - NCAATourneyCompactResults.csv: Game-by-game *NCAA tournament* results for all seasons of historical data since 1985 until 2017.
  
  - RegularSeasonDetailedResults.csv: Team-level box scores for many *regular seasons* of historical data, starting with the 2003 season.
  
  - NCAATourneyDetailedResults.csv: Team-level box scores for many *NCAA tournaments*, starting with the 2003 season until 2017
  
  - MasseyOrdinals_Prelim2018.csv: Rankings (e.g. #1, #2, #3, ..., #N) of teams from 2002-2003 *season* to 2018, under a large number of different ranking system methodologies.
  
  - Players_XXXX.csv: Players list  from 2010 to 2018.  Each player is assigned to team they belonged to. 
  
  - Events_XXXX.csv: Play-by-play event logs for almost all games from that season. Elapsed seconds, event type, points are recorded. This data does not include NCAA tournament 2018.
  
**(Test data)**  

  - 2018NCAATourneyCompactResults.csv: Game-by-game *NCAA tournament* results for 2018.
  
  - 2018NCAATourneyDetailedResults.csv: Team-level box scores for *NCAA tournament* 2018

```{r eval=FALSE}
RegularSeasonCompactResults=read.csv("data/PrelimData2018/RegularSeasonCompactResults_Prelim2018.csv")
NCAATourneyCompactResults=read.csv("data/DataFiles/NCAATourneyCompactResults.csv")
NCAATourneyCompactResults_2018=read.csv("data/DataFiles/2018NCAATourneyCompactResults.csv")

RegularSeasonDetailedResults=read.csv("data/PrelimData2018/RegularSeasonDetailedResults_Prelim2018.csv")
NCAATourneyDetailedResults=read.csv("data/DataFiles/NCAATourneyDetailedResults.csv")
NCAATourneyDetailedResults_2018=read.csv("data/DataFiles/2018NCAATourneyDetailedResults.csv")

MasseyOrdinals=read.csv("data/PrelimData2018/MasseyOrdinals_Prelim2018.csv")

Events_2010=read.csv("data/PlayByPlay_2010/Events_2010.csv")
Events_2011=read.csv("data/PlayByPlay_2011/Events_2011.csv")
Events_2012=read.csv("data/PlayByPlay_2012/Events_2012.csv")
Events_2013=read.csv("data/PlayByPlay_2013/Events_2013.csv")
Events_2014=read.csv("data/PlayByPlay_2014/Events_2014.csv")
Events_2015=read.csv("data/PlayByPlay_2015/Events_2015.csv")
Events_2016=read.csv("data/PlayByPlay_2016/Events_2016.csv")
Events_2017=read.csv("data/PlayByPlay_2017/Events_2017.csv")
Events_2018=read.csv("data/PrelimData2018/Events_Prelim2018.csv")

Players_2010=read.csv("data/PlayByPlay_2010/Players_2010.csv")
Players_2011=read.csv("data/PlayByPlay_2011/Players_2011.csv")
Players_2012=read.csv("data/PlayByPlay_2012/Players_2012.csv")
Players_2013=read.csv("data/PlayByPlay_2013/Players_2013.csv")
Players_2014=read.csv("data/PlayByPlay_2014/Players_2014.csv")
Players_2015=read.csv("data/PlayByPlay_2015/Players_2015.csv")
Players_2016=read.csv("data/PlayByPlay_2016/Players_2016.csv")
Players_2017=read.csv("data/PlayByPlay_2017/Players_2017.csv")
Players_2018=read.csv("data/PrelimData2018/Players_Prelim2018.csv")
```

Because Player_XXXX dataset had some strange value: "TEAM", we excluded this from data.

```{r eval=FALSE}
#Delete the rows that have "TEAM" in playername column
Players_2010=Players_2010[Players_2010$PlayerName!="TEAM",]
Players_2011=Players_2011[Players_2011$PlayerName!="TEAM",]
Players_2012=Players_2012[Players_2012$PlayerName!="TEAM",]
Players_2013=Players_2013[Players_2013$PlayerName!="TEAM",]
Players_2014=Players_2014[Players_2014$PlayerName!="TEAM",]
Players_2015=Players_2015[Players_2015$PlayerName!="TEAM",]
Players_2016=Players_2016[Players_2016$PlayerName!="TEAM",]
Players_2017=Players_2017[Players_2017$PlayerName!="TEAM",]
Players_2018=Players_2018[Players_2018$PlayerName!="TEAM",]
```

### Player score 
```{r eval=FALSE}
library(dplyr)
library(tidyr)
```
We struggled dealing with Player_XXXX data and Event_XXXX data because other data was based on every matchup but these data was based on the player and event. There were two solutions to solve this problem.

1. Aggregate events by each match and merge with other data, using each matchup as ID.

2. Give scores to each event, and aggregate events’ score by player and year. Then we can get each player's score in each year. Finally aggregate players' score by their belonging team. 

We thought choice 1 was too overfitting with each matchup and was lack of power to express the ranking of each team in general. Moreover, it was almost identical with Wscores and Lscores in the Compact results dataset. So we decide to use choice 2 to handle Player and Event data. First, we set scores to each event like below. 

- +1: assist, block, steal, reb_off, reb_def, reb_dead, made1_free, made2_tip, made2_dunk, made2_lay, made2_jump, made3_jump
- -1: turnover, foul_pers, foul_teach, miss1_free
- 0: timeout, timeout_tv, sub_in, sub_out, miss2_dunk, miss2_tip, miss2_lay, miss2_jump, miss3_jump

They are all based on our own subjective criteria. So there might be room for improvement.

**(Step 1) Give scores to each event, and aggregate events by player and year**

```{r eval=FALSE}
####get the score of each player every year#### 
##give the score to the player in the following way##
player_score=function(Events,Players){
  events=dplyr::full_join(Events,Players, by=c("Season","EventPlayerID"="PlayerID","EventTeamID"="TeamID"))
  events$events_score=ifelse(events$EventType%in% c("assist","block","steal","reb_off","reb_def","reb_dead","made1_free", "made2_dunk","made2_tip","made2_lay","made2_jump","made3_jump"),1, ifelse(events$EventType%in% c("turnover","foul_pers","foul_tech","miss1_free"),-1,0))
  events=
    group_by(events,PlayerName)%>%
    summarise(TotalScore=sum(events_score))
  return(events)
}
events2010=player_score(Events_2010,Players_2010)
events2011=player_score(Events_2011,Players_2011)
events2012=player_score(Events_2012,Players_2012)
events2013=player_score(Events_2013,Players_2013)
events2014=player_score(Events_2014,Players_2014)
events2015=player_score(Events_2015,Players_2015)
events2016=player_score(Events_2016,Players_2016)
events2017=player_score(Events_2017,Players_2017)
events2018=player_score(Events_2018,Players_2018)
```


```{r eval=FALSE}
#merge Players file and events 
player_score_2010=dplyr::
  full_join(Players_2010,events2010,by="PlayerName")
player_score_2011=dplyr::
  full_join(Players_2011,events2011,by="PlayerName")
player_score_2012=dplyr::
  full_join(Players_2012,events2012,by="PlayerName")
player_score_2013=dplyr::
  full_join(Players_2013,events2013,by="PlayerName")
player_score_2014=dplyr::
  full_join(Players_2014,events2014,by="PlayerName")
player_score_2015=dplyr::
  full_join(Players_2015,events2015,by="PlayerName")
player_score_2016=dplyr::
  full_join(Players_2016,events2016,by="PlayerName")
player_score_2017=dplyr::
  full_join(Players_2017,events2017,by="PlayerName")
player_score_2018=dplyr::
  full_join(Players_2018,events2018,by="PlayerName")
```

**(Step 2) Aggregate players' score by their belonging team**

```{r eval=FALSE}
#get the average player score of each team for every year 
team_score_2010=group_by(player_score_2010,TeamID)%>%
  summarise(Teamscore=sum(TotalScore))
team_score_2010$Season=2010
team_score_2010=team_score_2010[-dim(team_score_2010)[1],]
team_score_2011=group_by(player_score_2011,TeamID)%>%
  summarise(Teamscore=sum(TotalScore))
team_score_2011$Season=2011
team_score_2011=team_score_2011[-dim(team_score_2011)[1],]
team_score_2012=group_by(player_score_2012,TeamID)%>%
  summarise(Teamscore=sum(TotalScore))
team_score_2012$Season=2012
team_score_2012=team_score_2012[-dim(team_score_2012)[1],]
team_score_2013=group_by(player_score_2013,TeamID)%>%
  summarise(Teamscore=sum(TotalScore))
team_score_2013$Season=2013
team_score_2013=team_score_2013[-dim(team_score_2013)[1],]
team_score_2014=group_by(player_score_2014,TeamID)%>%
  summarise(Teamscore=sum(TotalScore))
team_score_2014$Season=2014
team_score_2014=team_score_2014[-dim(team_score_2014)[1],]
team_score_2015=group_by(player_score_2015,TeamID)%>%
  summarise(Teamscore=sum(TotalScore))
team_score_2015$Season=2015
team_score_2015=team_score_2015[-dim(team_score_2015)[1],]
team_score_2016=group_by(player_score_2016,TeamID)%>%
  summarise(Teamscore=sum(TotalScore))
team_score_2016$Season=2016
team_score_2016=team_score_2016[-dim(team_score_2016)[1],]
team_score_2017=group_by(player_score_2017,TeamID)%>%
  summarise(Teamscore=sum(TotalScore))
team_score_2017$Season=2017
team_score_2017=team_score_2017[-dim(team_score_2017)[1],]
team_score_2018=group_by(player_score_2018,TeamID)%>%
  summarise(Teamscore=sum(TotalScore))
team_score_2018$Season=2018
team_score_2018=team_score_2018[-dim(team_score_2018)[1],]

team_score=rbind(team_score_2010,team_score_2011)
team_score=rbind(team_score,team_score_2012)
team_score=rbind(team_score,team_score_2013)
team_score=rbind(team_score,team_score_2014)
team_score=rbind(team_score,team_score_2015)
team_score=rbind(team_score,team_score_2016)
team_score=rbind(team_score,team_score_2017)
team_score=rbind(team_score,team_score_2018)
write.csv(team_score,"team_score.csv")
```

### Merge data

Now the data are separated and so we need to combine them. First RegularSeasonCompactResults and NCAATourneyCompactResults were merged vertically because they had same column and no overlap. Merged data was named Compact_train. Likewise, RegularSeasonDetailedResults and NCAATourneyDetailedResults were combined and merged data was named Details_train. Then Compact_train and Details_train were joined. Please note that Compact_train and Detail_train do not necessarily have the same day number because Details_train starts from 2003 whereas Compact_train starts with 1985. This is why we used full join in this case to keep the all rows in both dataset. We did the same procedure to test data.

```{r eval=FALSE}
#Train data
Compact_train=rbind(RegularSeasonCompactResults, NCAATourneyCompactResults)
Details_train=rbind(RegularSeasonDetailedResults, NCAATourneyDetailedResults)
#I uesd left full because I want to keep the all row in both dataset
Results_train=dplyr::full_join(
  Compact_train,Details_train,
  by=c("Season","DayNum","WTeamID","LTeamID", "WScore","LScore","WLoc","NumOT"))

#Test data
Compact_test=NCAATourneyCompactResults_2018
Details_test=NCAATourneyDetailedResults_2018
Results_test=dplyr::full_join(
  Compact_test,Details_test,
  by=c("Season","DayNum","WTeamID","LTeamID", "WScore","LScore","WLoc","NumOT"))
```

We also changed MasseyOrdinals data from long data to wide data because it made it easier to combine with train data (Result_data). We merged twice for winner's rate and loser's rate. Consequently, Column 70T.x~ZAM.x is the rate for WTeam (Winning Team) and column 70T.y~ZAM.y is the rate for LTeam (Losing Team). 

We wanted to use MasseyOrdinals for test data too but it didn't have rate data for NCAA tournament in 2018. So, we decided to use the rate of DayNum=133 in 2018 as the rate for NCAA tournament in 2018. 

```{r eval=FALSE}
#spread the systemnae 
spread_rank=spread(MasseyOrdinals,key="SystemName", value="OrdinalRank")
dat_day_train=left_join(Results_train,spread_rank,
                        by=c("Season","DayNum"="RankingDayNum","WTeamID"="TeamID"))
dat_day_train=left_join(dat_day_train,spread_rank,
                        by=c("Season","DayNum"="RankingDayNum","LTeamID"="TeamID"))
#Column 70T.x~ZAM.x is the rate for WTeam
#Column 70T.y~ZAM.y is the rate for LTeam
train=dat_day_train
#Rate data only has the date before DayNum=134. This is why, I want to regard the rating of the DayNum=133 as the rate for the NCAATourney (DauNum>133)
spread_rank_for_test=spread_rank[spread_rank$RankingDayNum==114,]
dat_day_test=left_join(Results_test,spread_rank_for_test,
                       by=c("Season","WTeamID"="TeamID"))
dat_day_test=left_join(dat_day_test,spread_rank_for_test,
                       by=c("Season","LTeamID"="TeamID"))

#Drop the RankingDayNum variable
dat_day_test=dat_day_test[,c(-35,-200)]
test=dat_day_test
```

## Feature Engineering

### Part 1

We assumed that adding new features would provide a deeper understanding of a team's performance. That’s why we used indexes that were often used for analyzing basketball game. We refered to Kaggle[3] to get the new feature. The explanation of each variable is written in the code chunk as comments. Because the rate variables were too many, we decided to use only average of them.

```{r eval=FALSE}
#Points Winning/Losing Team
train$WPts=2*train$WFGM+train$WFGM3+train$WFTM
train$LPts=2*train$LFGM+train$LFGM3+train$LFTM
train$Pts_diff=train$WPts-train$LPts
#Calculate Winning/losing Team Possesion Feature
wPos=train$WFGA+train$WTO+0.44*train$WFTA-train$WOR
lPos=train$LFGA+train$LTO+0.44*train$LFTA-train$LOR
train$Pos_diff=train$WFGA-train$LFGA
#two teams use almost the same number of possessions in a game
#(plus/minus one or two - depending on how quarters end)
#so let's just take the average
train$Pos=(wPos+lPos)/2
#Offensive efficiency (OffRtg) = 100 x (Points / Possessions)
train$WOffRtg=100*(train$WPts/train$Pos)
train$LOffRtg=100*(train$LPts/train$Pos)
train$Off_diff=train$WOffRtg-train$LOffRtg
#Offensive efficiency (OffRtg) = 100 x (Points / Possessions)
train$WDefRtg = train$LOffRtg
train$LDefRtg = train$WOffRtg
#Net Rating = Off.Rtg - Def.Rtg
train$WNetRtg=train$WOffRtg-train$WDefRtg
train$LNetRtg=train$LOffRtg-train$LDefRtg
train$Net_diff=train$WNetRtg-train$LNetRtg
#Assist Ratio : Percentage of team possessions that end in assists
train$WAstR=100*train$WAst/(train$WFGA + 0.44*train$WFTA+ train$WAst + train$WTO)
train$LAstR=100*train$LAst/(train$LFGA + 0.44*train$LFTA+ train$LAst + train$LTO)
train$AstR_diff=train$WAstR-train$LAstR
#Turnover Ratio: Number of turnovers of a team per 100 possessions used.
#(TO * 100) / (FGA + (FTA * 0.44) + AST + TO)
train$WTOR=100 * train$WTO / (train$WFGA + 0.44*train$WFTA + train$WAst + train$WTO)
train$LTOR=100 * train$LTO / (train$LFGA + 0.44*train$LFTA + train$LAst + train$LTO)
train$TOR_diff=train$WTOR-train$LTOR
#The Shooting Percentage : Measure of Shooting Efficiency (FGA/FGA3, FTA)
train$WTSP=100 * train$WPts / (2 * (train$WFGA + 0.44*train$WFTA))
train$LTSP=100 * train$LPts / (2 * (train$LFGA + 0.44*train$LFTA))
train$TSP_diff=train$WTSP-train$LTSP
#eFG% : Effective Field Goal Percentage adjusting for the fact that 3pt shots are more valuable 
train$WeFGP=(train$WFGM + 0.5 *train$WFGM3) / train$WFGA
train$LeFGP=(train$LFGM + 0.5 *train$LFGM3) / train$LFGA
train$eFGP_diff=train$WeFGP-train$LeFGP
#FTA Rate : How good a team is at drawing fouls.
train$WFTAR = train$WFTA / train$WFGA
train$LFTAR = train$LFTA / train$LFGA
train$FTAR_diff=train$WFTAR-train$LFTAR
#OREB% : Percentage of team offensive rebounds
train$WORP = train$WOR / (train$WOR + train$LDR)
train$LORP = train$LOR / (train$WOR + train$LDR)
train$ORP_diff=train$WORP-train$LORP
#DREB% : Percentage of team defensive rebounds
train$WDRP = train$WDR / (train$WDR + train$LOR )
train$LDRP=train$LDR / (train$LDR + train$WOR )
train$DRP_diff=train$WDRP-train$LDRP
#REB% : Percentage of team total rebounds
train$WRP=(train$WDR + train$WOR) / (train$WDR + train$WOR + train$LDR + train$LOR)
train$LRP=(train$LDR + train$LOR) / (train$WDR + train$WOR + train$LDR + train$LOR)
train$RP_diff=train$WRP-train$LRP

```

```{r eval=FALSE}
# use the average ranking 
train$Wranking=apply(train[,35:198],1,function(x) mean(x,na.rm=TRUE))
train$Lranking=apply(train[,199:362],1,function(x) mean(x,na.rm=TRUE))
train=train[,c(-9:-362)]
colnames(train)[7]="Loc"
sub_train=train
```

### Part 2

Now we have the data like model1 below. This seemed to be good enough to analyze data but we tried to enrich data. If we reverse the winning team and losing team and add this data to the existing data, this will make the data more robust because this will allow for each variable to take more variation. Moreover in the model1, the outcome is all the same among all samples  (i.e. outcome is all win (or lose)) and this will cause the model to return only one value. By enriching data, we can avoid those problems. 

("enrich.png")

```{r eval=FALSE}
#In this case, winner team is named as TeamID1 and the result is 1
train_1=train
colnames(train_1)[3]="TeamID1"
colnames(train_1)[5]="TeamID2"
colnames(train_1)[4]="Team1_score"
colnames(train_1)[6]="Team2_score"
for(i in c(9,14,17,19,22,25,28,31,34,37,40,43,46)){
  colnames(train_1)[i]=paste0("Team1_",substr(colnames(train_1)[i],
                                              2,nchar(colnames(train_1)[i])))
}
for(i in c(10,15,18,20,23,26,29,32,35,38,41,44,47)){
  colnames(train_1)[i]=paste0("Team2_",substr(colnames(train_1)[i],
                                              2,nchar(colnames(train_1)[i])))
}
winners=train_1
winners$Result=1.0
```

```{r eval=FALSE}
#In this case, winner team is named as TeamID1 and the result is 1
train_2=train
colnames(train_2)[3]="TeamID2"
colnames(train_2)[5]="TeamID1"
colnames(train_2)[4]="Team2_score"
colnames(train_2)[6]="Team1_score"
for(i in c(9,14,17,19,22,25,28,31,34,37,40,43,46)){
  colnames(train_2)[i]=paste0("Team2_",
                              substr(colnames(train_2)[i],
                                     2,nchar(colnames(train_2)[i])))
}
for(i in c(10,15,18,20,23,26,29,32,35,38,41,44,47)){
  colnames(train_2)[i]=paste0("Team1_",
                              substr(colnames(train_2)[i],
                                     2,nchar(colnames(train_2)[i])))
}
train_2[,c(11,12,16,21,24,27,30,33,36,39,42,45)]=-
  train_2[,c(11,12,16,21,24,27,30,33,36,39,42,45)]
train_2$Loc=ifelse(train_2$Loc=="A","H","A")
losers=train_2
losers$Result=0.0
```

```{r eval=FALSE}
#Combine them
train=rbind(winners,losers)
```

Next, we combined train data we made with team_score data which we made at player score section. 

```{r eval=FALSE}
#Combine with team score
train=dplyr::left_join(train,team_score,by=c("Season","TeamID1"="TeamID"))
train=dplyr::left_join(train,team_score,by=c("Season","TeamID2"="TeamID"))
```

```{r eval=FALSE}
#Drop the variables we don't use
train_X=train[,c(-2,-4,-6,-8,-48)]
train_y=train[,48]
colnames(train_X)[44]="player_score_1"
colnames(train_X)[45]="player_score_2"
```

### Part 3

Let's move onto test data. We wanted to make features of test data but we had to be careful when making them. We can't use the features we made in part1 because it will cause data leakage.  For example, we can easily know that it's impossible to get the details of matchup, such as winning/ losing points and winning/losing team possession before the match is actually done. So we have to delete the variables that we can't know before the matchup actually starts.  

```{r eval=FALSE}
#use the average ranking 
test$Wranking=apply(test[,35:198],1,function(x) mean(x,na.rm=TRUE))
test$Lranking=apply(test[,199:362],1,function(x) mean(x,na.rm=TRUE))
# remove the value from WFGM~LPF from test data because otherwise it will cause data leakage
test=test[,c(-8:-362)]
```
  
However, we wanted to use the expected values of variables like winning/ losing points and winning/losing team possession as predictors. How can we get the expected value for test data? Here. we developed some algorithm like below. 
  
  1. when there was the same match up before

Let's illustrate the following table. The first row in test data is Team10 vs Team7. If we look at the train data, we can find the same matchup in train data. In that case, we take the average of same matchup in training data for every feature like poins, possesion, offensive efficiency and so on. This idea is based on we can expect the same values as the previous same match.

  2. when there was no same matchup before

Then how about when there was no same matchup before? In that case, we take an average of features of teams with similar ranking in train data. In the below table, the matchup Team 10 vs Team12 didn't occur in train data and so we took the average of Team 10 vs Team 23 and Team10 vs Team 90 because the opponents' ranking is similar with Team12's ranking.  

("test_feature.png")

```{r eval=FALSE}
library(class)
#Get the mean of every feature calculated from the previous same pair of match
#If there was not the exactly the same match, we calculate the mean of similar match
  
create_train_feature=function(dat,colum){
    #Get the average of same matchup
    a=dat%>%group_by(TeamID1,TeamID2)%>%summarise(
      mean=mean(.data[[colnames(dat)[colum]]],na.rm=TRUE))
    #change the colum name
    colnames(a)[3]=colnames(sub_train)[colum+4]
    #combine with test data
    test=dplyr::left_join(test,a,by=c("WTeamID"="TeamID1","LTeamID"="TeamID2"))
    #specify where they don't have the same matchup in train data 
    naindex=which(is.na(test[colnames(sub_train)[colum+4]]))
    #Losing teamID of this matchup
    LTeamID=test$LTeamID[naindex]
    #Wining teamID of this matchup
    WTeamID=test$WTeamID[naindex]
    #Get the ranking of losing team
    Lranking=sapply(LTeamID, function(x) mean(dat$Team2_ranking[dat$TeamID2==x],na.rm=TRUE))
    knn_target=rep(NA,length(LTeamID))
    count=1
    for(i in WTeamID){
      #list of the team that Team1 had battled in train data
      ranking_list=dat[dat$TeamID1==i,"Team2_ranking"]
      #list of the value of desired variable that Team1 scored in train data  
      corresponding_target=dat[dat$TeamID1==i,colnames(dat)[colum]]
      #make the dataframe
      train=data.frame(rank=ranking_list,target=corresponding_target)
      train=drop_na(train)
      #use the k nearest neighbors to get the average of features of teams with similar ranking in train data
      knn_target[count]=knn(train=train$rank, test =Lranking[count] , cl = train$target, k = 5)
      count=count+1
    }
    count=1
    for(i in naindex){
      test[i,colnames(sub_train)[colum+4]]=knn_target[count]
      count=count+1
    }
    return(test)
}

#apply create_test_feature to the test data
for(i in 5:41){
    test=create_train_feature(dat=train_X,colum =i)
  }

```

We applied the same process as part 2 to test data and enriched the data.

```{r eval=FALSE}
#combine testdata with team_score
  test=dplyr::left_join(test,team_score,
                        by=c("Season","WTeamID"="TeamID"))
  test=dplyr::left_join(test,team_score,
                        by=c("Season","LTeamID"="TeamID"))
  test=test[,c(-2,-4,-6)]
  colnames(test)[4]="Loc"
```

```{r eval=FALSE}
  winner=test
  colnames(winner)[2]="TeamID1"
  colnames(winner)[3]="TeamID2"
  
  for(i in c(5,7,12,15,17,20,23,26,29,32,35,38,41)){
    colnames(winner)[i]=paste0("Team1_",substr(colnames(winner)[i],
                                               2,nchar(colnames(winner)[i])))
  }
  for(i in c(6,8,13,16,18,21,24,27,30,33,36,39,42)){
    colnames(winner)[i]=paste0("Team2_",substr(colnames(winner)[i],
                                               2,nchar(colnames(winner)[i])))
  }
  winner$Result=1.0
  loser=test
  colnames(loser)[2]="TeamID2"
  colnames(loser)[3]="TeamID1"
  for(i in c(5,7,12,15,17,20,23,26,29,32,35,38,41)){
    colnames(loser)[i]=paste0("Team2_",substr(colnames(loser)[i],
                                              2,nchar(colnames(loser)[i])))
  }
  for(i in c(6,8,13,16,18,21,24,27,30,33,36,39,42)){
    colnames(loser)[i]=paste0("Team1_",substr(colnames(loser)[i],
                                              2,nchar(colnames(loser)[i])))
  }
  loser[,c(9,10,14,19,22,25,28,31,34,37,40,43)]= -winner[,c(9,10,14,19,22,25,28,31,34,37,40,43)]
  loser$Loc=ifelse(loser$Loc=="A","H","A")
  loser$Result=0.0
  test=rbind(winner,loser)
  colnames(test)[44]="player_score_1"
  colnames(test)[45]="player_score_2"
  test_X=test[,-46]
  test_y=test[,46]
  test_X=test_X[,c(1:4,7:43,5,6,44,45)]
```


```{r eval=FALSE}
test_X$Team_ranking_diff=
  test_X$Team1_ranking-test_X$Team2_ranking
test_X$player_score_diff=
  test_X$player_score_1-test_X$player_score_2
train_X$Team_ranking_diff=
  train_X$Team1_ranking-train_X$Team2_ranking
train_X$player_score_diff=
  train_X$player_score_1-train_X$player_score_2
```


```{r eval=FALSE}
train=cbind(train_X,train_y)
train_final=drop_na(train)
test=cbind(test_X,test_y)
colnames(test)[48]="train_y"
test_final=test
```

```{r eval=FALSE}
write.csv(train_final,"train_final.csv")
write.csv(test_final,"test_final.csv")
```
  

### Part 4
  Finally, we tried PCA because the data has many variables and might cause collinearity. We first extracted the numerical variables and then performed PCA. As you can see in the plot, most of the variation was explained after 5th components. I used different number of components in the following Statistical Modeling part and it turned out that there was not much big difference about the choice of number of components after 5.

```{r}
  train_final=read.csv("train_final.csv")[,-1]
  test_final=read.csv("test_final.csv")[,-1]
  train_final_for_pca=train_final[,c(-2,-3,-4,-7,-48)]
  test_final_for_pca=test_final[,c(-2,-3,-4,-7,-48)]
  pca=prcomp(train_final_for_pca,scale. = TRUE)
  plot(pca,type="l")
```

```{r}
  train_pca=pca$x[,1:9]
  test_pca <- predict(pca, newdata =test_final_for_pca )
  test_pca=data.frame(test_pca[,1:9])
```


## Statistical Modeling

  Data processing was hard for this dataset and during processing, we used knn to get the adequate predictors in test data. So we can say that data process itself is the statistical modeling and has the power to predict the result of matchup in NCAA tournament in 2018. That is, points difference we predicted between Team1 and Team2 is itself the good indicator of the result. If it's more than zero, it means that Team1 is winning team and if it's less than zero, Team2 is winning team. The accuracy of this simple model was 86.56% and this was quite good. 
  
```{r}
  #Pts
  result_from_pts=ifelse(test_final$Pts_diff>0,1,0)
  table(result_from_pts,test_final[,48])
  mean(result_from_pts==test_final[,48])
```
  
  We also tried logistic regression, Linear Discriminant Analysis (LDA), Quadratic Discriminant Analysis (QDA), Random Forest and Neural Network to solve this classification problem (win or lose) and wanted to see applying other statistical methods improved the accuracy or not. 

First we used logistic regression. Accuracy was 82.09%.

```{r,echo=FALSE,message=FALSE}
  suppressMessages(c(library(dplyr),
  library(tidyr),
  library(MASS),
  library(randomForest)))
```

```{r}
  #logistic regression
  y=train_final[,48]
  train_pca=data.frame(cbind(train_pca,y))
  fit=glm(y~.,data = train_pca,family = binomial(link=logit))
  predicted <- predict(fit, test_pca, type="response")
  result.pred = rep(0, length(predicted))
  result.pred[predicted > .5] = 1
  table(result.pred,test_final[,48])
  mean(result.pred==test_final[,48])
```

Next we used LDA, whose accuracy was 80.59%. 

```{r}
  #LDA
  dig.lda=lda(train_pca[,1:9],y)
  Ytest.pred.lda=predict(dig.lda, test_pca[,1:9])$class
  table(test_final[,48],Ytest.pred.lda)
  mean(test_final[,48]==Ytest.pred.lda)
```

QDA was also applied but it didn't work well. 

```{r}
  #QDA
  dig.qda=qda(train_pca[,1:9],y)
  Ytest.pred.qda=predict(dig.qda, test_pca[,1:9])$class
  table(test_final[,48],Ytest.pred.qda)
  mean(test_final[,48]==Ytest.pred.qda)
```

Random forest returned the same accuracy as simple model using only points difference. We referred to lecture notes and set the parameter. Number of tree is 1500, which is large enough. Number of variables considered at each split is 3 based on the criteria $\sqrt{p}$. Node size is 1 because this problem is classification. 

```{r}
  #Random Forest
  rf.fit = randomForest(train_final_for_pca, as.factor(y), ntree = 1500, mtry = 5, nodesize = 1, sampsize = 500)
  pred = predict(rf.fit, test_final_for_pca)
  table(test_final[,48],pred)
  mean(test_final[,48]==pred)
```

We also tried Neural Network, especially fully connected layer and drop out. Activation function was relu for the first layer and sigmoid for output layer. Unit size was 32 for the first layer and 1 for output layer. We also used l2 kernel regularizer to avoid overfit.   

```{r}
#Neural Network
train_mean=apply(train_final_for_pca, 2, FUN=mean)
train_for_neural=scale(train_final_for_pca,center = train_mean, scale = FALSE)
test_for_neural=scale(test_final_for_pca,center = train_mean, scale = FALSE)

library(keras)
k_clear_session()
model <- keras_model_sequential() %>% 
    layer_dense(units = 32, activation = "relu", kernel_regularizer = regularizer_l2(0.001),
                input_shape = dim(train_for_neural)[2]) %>% 
    layer_dropout(rate = 0.5) %>%
    layer_dense(units = 1,kernel_regularizer = regularizer_l2(0.001), activation = "sigmoid") 

model %>% compile(
    optimizer = optimizer_rmsprop(lr=0.001), 
    loss = "binary_crossentropy", 
    metrics = c("accuracy")
)
```

We used 20% of train data as validation data and 80% as train data. The accuracy was 76.12% and this was not as good as logistic regression and random forest but there might be room for improvement by changing the architecture of network.

```{r}
set.seed(100)

index <- sample(dim(train_for_neural)[1],10000,replace = FALSE)
x_val=train_for_neural[index,]
x_train=train_for_neural[-index,]

y_val <- y[index]
y_train <- y[-index]

num_epochs <- 30
history=model %>% fit(x_train, y_train,
                epochs = num_epochs, batch_size = 128,validation_split = 0.2)
plot(history)
results <- model %>% evaluate(test_for_neural, test_final[,48])
results
table(predict_classes(model, test_for_neural),test_final[,48])
```

# Conclusion

The best accuracy was achieved by both simple model using Pts difference: point difference between winning team and losing team and random forest. We can consider that this first one is part of the random forest. It is equivalent to the model where only points difference was considered at split in random forest. Consequently, it turned that our algorithm that predicted the expected value of points difference performed very well.

From this analysis, we found the model that could possibly predict the winning teams and the losing teams. Something that we can improve with this analysis is to actually incorporate *seeding* variable (as that is important aspect to consider to see who plays with who) to possibly predict who the real winner was after running through real simulations. At this point, we can see *who* might win. 

# Sources Cited

[1] https://www.wikipedia.org/wiki/NCAA_Division_I_Men's_Basketball_Tournament
[2] https://www.kaggle.com/c/mens-machine-learning-competition-2018/data
[3] https://www.kaggle.com/lnatml/feature-engineering-with-advanced-stats

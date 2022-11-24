# Complete Project 3: European King: Boxing Clasification in Football
# Scenario
You are a junior data analyst working for a business intelligence consultant. You have been at your job for six months, and your boss feels you are ready for more responsibility. He has asked you to lead a project for a brand new client — this will involve everything from defining the business task all the way through presenting your data-driven recommendations. You will choose the topic, ask the right questions, identify a fresh dataset and ensure its integrity, conduct analysis, create compelling data visualizations, and prepare a presentation.
# Main question to answer:
1. What type of company does your client represent, and what are they asking you to accomplish?
2. What are the key factors involved in the business task you are investigating?
3. What type of data will be appropriate for your analysis?
4. Where will you obtain that data?
5. Who is your audience, and what materials will help you present to them effectively?
# 1.0 Ask
I have chosen this database because it fit me for my analysis. What I want to do is to take the boxing championship model to soccer. Every time someone defeats the champion, he will take his crown and will be the 'King of Europe' until he is defeated. There only 2 rules:

The first King of Europe is Real Madrid because they won the previous year's Champions League from this database (2001-02)
By the time a new King is is crowned (New Champions League Winner), he will automatically be the new king of europe.
**No more rules, let's get the game started!**

# 2.0 Prepare
## 2.1 Data preparation
```
library(tidyverse)
library(dplyr)
library(tidyr)
library(ggplot2)
library(lubridate)
library(forcats)
library(viridis)
```
```
matches <- read.csv('../input/european-soccer-data/Full_Kaggle_Dataset.csv')
```
```
str(matches)
```
We will have to make some adjustments to the columns. In excel I have seen that there are matches without results, mainly because of Covid-19, we proceed to eliminate those records since they were not played.
```
matches <- matches %>% drop_na(Home_Score) %>% drop_na(Away_Score)
str(matches)
```
Let us now set up data types for the Date and integers to the number of goals: (Time get me into an error because it appends me the current date, so I'm going to use it as chr)
```
matches$Date <- 
    strptime(matches$Date, '%d/%m/%Y')

matches <- matches %>% mutate_at(c('Home_Score', 'Away_Score', 'Home_Score_AET', 'Away_Score_AET', 
                                   'Home_Penalties', 'Away_Penalties', 'Home_Points', 'Away_Points' ), as.integer)

str(matches)
```
```
matches <- dplyr::arrange(matches, Date)
str(matches)
```
Since we've got our table ready, it's turn to set the king of europe column:

# 3.0 Process
First we make a column filled with NAs:
```
matches$Europe_King <- c('')
str(matches)
```
```
for(i in 1:nrow(matches)) {
  if(i == 1) {
    matches[i, 'Europe_King'] <- 'REAL MADRID'
  }else if(matches[i, 'Round'] == 'final' &
          matches[i, 'Competition'] == 'uefa-champions-league' & 
          matches[i, 'Home_Points'] > 0) {
    matches[i, 'Europe_King'] <- matches[i, 'Home_Team']
  }else if(matches[i, 'Round'] == 'final' &
          matches[i, 'Competition'] == 'uefa-champions-league' & 
          matches[i, 'Away_Points'] > 0) {
    matches[i, 'Europe_King'] <- matches[i, 'Away_Team']
  }else if(matches[i, 'Home_Team'] == matches[i-1, 'Europe_King'] &
          matches[i, 'Home_Points'] == 0) {
    matches[i, 'Europe_King'] <- matches[i, 'Away_Team']
  }else if(matches[i, 'Away_Team'] == matches[i-1, 'Europe_King'] &
          matches[i, 'Away_Points'] == 0) {
    matches[i, 'Europe_King'] <- matches[i, 'Home_Team']
  }else if(is.na(matches[i, 'Home_Points']) == TRUE |
          is.na(matches[i, 'Away_Points']) == TRUE) {
    matches[i, 'Europe_King'] <- matches[i-1, 'Europe_King']
  }else{
    matches[i, 'Europe_King'] <- matches[i-1, 'Europe_King']
  }
}
str(matches)
```
# 4.0 Analysis
## 4.1 Which team is the current king of europe and which team has been the king of europe for the most games?
Oh! My favourite part, let´s check who is the **King of Europe**
```
tail((matches$Europe_King), n=1)
```
And the team most matches as Europe King it's...
```
matches %>%
    group_by(Europe_King) %>%
    summarise(count=n()) %>%
    arrange(desc(count)) %>%
    top_n(10) %>%

ggplot(aes(x = fct_reorder(Europe_King, count), y = count, fill=count)) +
    geom_col() +
    coord_polar()+
    labs(title='Fig. 01: Total games as King of Europe', y='Games',x='', fill='Matches')+
    scale_fill_viridis(direction = -1)
```
That's a little less surprising, since WE've win so many titles among those years! (Real fan here).

European trophies in the period:
```
matches %>%
    group_by(Europe_King) %>%
    filter(Round=='final') %>%
    filter(Competition=='uefa-champions-league') %>%
    summarise(count=n()) %>%

ggplot(aes(x = fct_reorder(Europe_King, count), y = count, fill=count)) +
    geom_col() +
    coord_flip() +
    labs(title='Fig. 02: UEFA Champions League Titles', x='Team', y = 'Titles', fill='Titles')+
    scale_fill_viridis(direction = -1)
```
## 4.2. How many kings have been per season? Which teams have been champions for the longest time per season?
```
matches %>%
  group_by(season) %>%
  summarise(kings=n_distinct(Europe_King)) %>%


ggplot(aes(season, kings, fill=kings)) +
    geom_col() +
    geom_smooth(alpha=0.3, col='red')+
    labs(title='Fig. 03: Distinct Kings of Europe per year', x='Year', y = 'Europe Kings', fill='Europe Kings')+
    scale_fill_viridis(direction = -1)
```
Oh! i think it's coming together in fewer hands... I enjoy much more when team that doesn't play in europe with the crown...
```
matches %>%
    group_by(Europe_King, season) %>%
    summarise(count=n()) %>%
    arrange(desc(count)) %>%
    head(3) %>%

ggplot(aes(x = fct_reorder(Europe_King, count, .desc=TRUE), y = count, fill=count)) +
    geom_col() +
    labs(title='Fig. 04: Most matches as King of Europe in a single season', x='Team', y = 'Matches as King', fill='Matches')+
    scale_fill_viridis(direction = -1)
```
YES!! Winner's again. Now lets see the stastics but for the best 75 seasons:
```
matches %>%
    group_by(Europe_King, season) %>%
    summarise(count=n()) %>%
    arrange(desc(count)) %>%
    head(75) %>%

ggplot(aes(x=Europe_King, y=season, fill=count)) + #x = fct_reorder(Europe_King, count)
    geom_raster() +
    coord_flip() +
    labs(title='Fig. 05: Most matches as King of Europe by season. Top 75 Kings.', y='Season', x = 'Team', fill='Matches')+
    scale_fill_viridis(direction = -1)
```
 love doing these visualizations and playing with the data... There are a lot of teams out there that are not the 'big' ones, what a thrill!
 
 ## 4.3 Country statistics. Most Kings of Europe by Country and Season
 To get the country of each team, I have added the number of home games played by the kings of europe and eliminated those played in european competition.

I have saved it as a dataframe containing the countries of each team for future calculations:
```
team_country <-
matches %>% 
    group_by(Home_Team, Country, Europe_King) %>%
    count(Country, Europe_King)  %>%
    filter(Home_Team==Europe_King) %>%
    filter(Country!='europe-uefa')


team_country <- team_country[, -c(3:4)]
team_country <- ungroup(team_country)
colnames(team_country)[1] <- c("Team")

str(team_country)
```
```
team_country %>%
    group_by(Country) %>%
    summarise(count=n()) %>%

ggplot(aes(x = fct_reorder(Country, count), y = count, fill=count)) +
    geom_col() +
    #coord_polar()+
    labs(title='Fig. 06: Total matches as King of Europe by Country', x= ' ', y='Kings per Country', fill='Kings per Country')+
    scale_fill_viridis(direction = 1)
``` 
```
kigs_season <-
matches %>%
    group_by(season, Europe_King) %>%
    summarise(count=n_distinct(Europe_King)) 

king_season_country <- merge(x=kigs_season, y=team_country, by.x=c('Europe_King'),by.y=c('Team'))

king_season_country %>%
ggplot(aes(season, count, fill=Country)) +
    geom_col() +
    labs(title='Fig. 07: Total Kings of Europe by Season and Country', x= 'Year', y='Total Kings', fill='Total Kings')+
    scale_fill_viridis_d(direction = -1)
```
# 5.0 Conclusion
This database has helped me to put into practice everything I know, since it has been very difficult for me to get the current champion through iteration with the previous columns, I have taken a lot of knowledge from this project.

As for the countries, it has not been easy either because there were many matches in which the champion did not play and it has been more difficult for me to find out which country each winner belongs to.

As for the project, I recommend the other countries and teams to get their act together because both Real Madrid and Spain have achieved the best results, only closely followed by our eternal rival, FC. Barcelona.

As for Lille... just count the seconds to get knocked out by Real!!






 




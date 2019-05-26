# MLB-Postseason-Simulator
 
## Introduction
This project simulates 2016 Major Baseball League postseason using Monte Carlo algorithm and calculate winning probability for each of the 8 participants. The goal is to find out whether we can use statistics from regular season to infer the outcome of postseason, in the other words, the team with the most wins in regular season has the highest probability to win the world series title?   
The methodologies and assumptions of this project are based off [Rudelius, Thomas](https://econpapers.repec.org/article/bpjjqsprt/v_3a8_3ay_3a2012_3ai_3a1_3an_3a10.htm)'s research published on *Journal of Quantitative Analysis in Sports (2012)*

## Modeling Framework
**Plate Appearance and Player Stats**  

Model input data was collected and calculated from BaseballReference.com. We used statistic from 2016 regular season to model batting and pitching performances for the starting lineups of each team in the postseason.  
Each batters would have a multinomial distribution that consists of 5 events, *single, double, triple, home run and walk*, and each pitchers would have a multinomial distribution of these 5 events that are given away. Since the chance for a pitcher getting a double or triple in a matchup is out of pitcher's control and mostly depends on the speed of the batter or the defensive skill of the fielders, a conditional probability of league average is baked in, for example, a probability of a pitcher getting a double is calculated as follow,

<a href="https://www.codecogs.com/eqnedit.php?latex=\large&space;P(2B)_{P}=&space;P(2B&space;|&space;2B\cup&space;3B)_{LgAvg}&space;\cdot&space;P(2B&space;\cup&space;3B)_{P}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\large&space;P(2B)_{P}=&space;P(2B&space;|&space;2B\cup&space;3B)_{LgAvg}&space;\cdot&space;P(2B&space;\cup&space;3B)_{P}" title="\large P(2B)_{P}= P(2B | 2B\cup 3B)_{LgAvg} \cdot P(2B \cup 3B)_{P}" /></a>

After created models for each player, we then applied **Log5 Method**, a variant of Bayes' formula developed by James, Bill in *Baseball Abstract (1981)* to calculate the outcome distribution for each plate appearance, the formula is represented as follow (e.g. for single hit):

<a href="https://www.codecogs.com/eqnedit.php?latex=\large&space;P(1B)=&space;\frac{(P(1B)_{Batter}\cdot&space;P(1B)_{Pitcher})&space;/&space;P(1B)_{LgAvg}}{(P(1B)_{Batter}\cdot&space;P(1B)_{Pitcher})&space;/&space;P(1B)_{LgAvg}&plus;(1-P(1B)_{Batter})\cdot&space;(1-P(1B)_{Pitcher})&space;/&space;(1-P(1B)_{LgAvg})}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\large&space;P(1B)=&space;\frac{(P(1B)_{Batter}\cdot&space;P(1B)_{Pitcher})&space;/&space;P(1B)_{LgAvg}}{(P(1B)_{Batter}\cdot&space;P(1B)_{Pitcher})&space;/&space;P(1B)_{LgAvg}&plus;(1-P(1B)_{Batter})\cdot&space;(1-P(1B)_{Pitcher})&space;/&space;(1-P(1B)_{LgAvg})}" title="\large P(1B)= \frac{(P(1B)_{Batter}\cdot P(1B)_{Pitcher}) / P(1B)_{LgAvg}}{(P(1B)_{Batter}\cdot P(1B)_{Pitcher}) / P(1B)_{LgAvg}+(1-P(1B)_{Batter})\cdot (1-P(1B)_{Pitcher}) / (1-P(1B)_{LgAvg})}" /></a>

League average probability was calculated with American League stats only because National League lacks of designated hitter would cause average batting performance skewed.  
Each plate appearance now has an outcome of either *single, double, triple, home run, or walk*. But there is also a possibility of *error* occurred on the field. Since errors are hardly accountable for the pitcher or the batter's ability, we used conditional probability of an error given the batter doesn't hit or walk, as a constant, which was approximated by the number of errors divided by number of put-outs in the regular season.

**Player Lineups**

The batting order in each team was decided by the number of starts in the regular season, for example, the first lineup would be the player who had the most starts in the first lineup in the regular season.  
Since pitcher's batting stats is quite limited and skewed, we used the average of ninth lineup in National League to replace the pitchers in National League.  
The pitching rotation order is decided by the 4 most prolific starting pitchers plus one relief pitcher. Relief pitcher would take over every games since the 7th inning to the end. The stats of relief pitchers were averaged from the 5 most prolific relief pitchers of each team. 

**Base Counter**

To simplify the rule of how runners move on the bases when the batter hit in a given plate appearance, logic stated at below,
 - *Single* or *Error*: every runners advance one base
 - *Double*: every runners advance two bases
 - *Triple*: every runners advance three based
 - *Home run*: every runners return to home base
 - *Walk*: runner is pushed to next base when previous base is occupied 

Stolen base and sacrifice bunt are neglected from the model since it's very difficult to determine when a specific team would call the action.

## Simulation Results 
Every series in 2016 postseason were simulated 200 times to obtain the winning probability for each team, division series is best of 5 while league series and world series are best of 7.  
Below is the schedule of 2016 postseason. 

|American League  |National League  |
|--|--|
|Rangers *vs* Blue Jays|Dodgers *vs* Nationals|
|Indians *vs* Red Sox  |Giants *vs* Cubs|


After simulated for 28 possible team matchups (6 possible matchups for each league and 16 possible matchups for world series), we were able to know the winning probabilities in each series for each team.  

|AL teams  |P(Win LDS)|P(Win LCS)|P(Win WS)|
|--|--|--|--|
|Rangers  |0.47|0.151|0.042|
|Blue Jays|0.53|0.212|0.083|
|Indians  |0.36|0.188|0.064|
|Red Sox  |0.64|0.449|0.245|

|NL teams  |P(Win LDS)|P(Win LCS)|P(Win WS)|
|--|--|--|--|
|Dodgers  |0.42|0.186|0.091|
|Nationals|0.58|0.268|0.145|
|Giants   |0.25|0.093|0.038|
|Cubs     |0.75|0.453|0.292|


The probability of winning LCS for a particular team is calculated from the probability of that team winning LDS multiply by the probability of beating next possible opponent weighted by the probability of opponent winning LDS, for example, *P(Rangers wins LCS) = P(Rangers wins LDS) x [P(Indians wins LDS) x P(Rangers beats Indians) + P(Red Sox wins LDS) x P(Rangers beats Red Sox)]*. The probability for winning WS follows this logic.

## Conclusion
Let's look back the question we would like to answer at the beginning of the project, the team with the most wins in regular season has the highest probability to win the world series title?  
From our simulation result Cubs has the highest rate of 29.2% to win the world series. In reality Cubs did win the 2016 world championship and also won the most games in the regular season (table at below). Seems regular season stats pretty much told the fates? But when we look closer at who wins the American LCS from the simulation result, we see Red Sox has the highest winning rate, who actually only won 93 games in regular season, behind Rangers and Indians. Even though Rangers have second most wins in the all teams, they are predicted to lose to Blue Jays in the first round.    

|Teams  |Wins  |
|--|--|
|Cubs     |103|  
|Rangers  |95 |
|Nationals|95 |
|Indians  |94 |
|Red Sox  |93 |
|Dodgers  |91 |
|Blue Jays|89 |
|Giants   |87 |

One fair explanation for this is the number of wins in the regular season of Rangers (95) is too close to Indians (94) and Res Sox (93), which gives some extent of uncertainty. In fact, Rangers's 47% LDS winning rate isn't even significant enough (based on 200 times simulation, one standard error equals to 3.5%), not to mention errors are compound in the next series.   

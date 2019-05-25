# MLB-Postseason-Simulator
 
## Introduction
This project simulates 2016 Major Baseball League postseason using Monte Carlo algorithm and calculate winning probability for each of the 8 participants. The goal is to find out whether we can use statistics from regular season to infer the outcome of postseason, in the other words, the team with the most wins in regular season has the highest probability to win the world series title?   
The methodologies and assumptions of this project are based off [Rudelius, Thomas](https://econpapers.repec.org/article/bpjjqsprt/v_3a8_3ay_3a2012_3ai_3a1_3an_3a10.htm)'s research published on *Journal of Quantitative Analysis in Sports (2012)*

## Modeling Framework

**Plate Appearance Outcome and Player Stats**  

Model input data was collected and calculated from BaseballReference.com. We used statistic from 2016 regular season to model batting and pitching performances for the starting lineups of each team in the postseason.  
Each batters would have a multinomial distribution that consists of 5 events, *single, double, triple, home run and walk*, and each pitchers would have a multinomial distribution of these 5 events that are given away. Since the chance for a pitcher getting a double or triple in a matchup is out of pitcher's control and mostly depends on the speed of the batter or the defensive skill of the fielders, a conditional probability of league average is baked in, for example, a probability of a pitcher getting a double is calculated as follow,

<a href="https://www.codecogs.com/eqnedit.php?latex=\large&space;P(2B)_{P}=&space;P(2B&space;|&space;2B\cup&space;3B)_{LgAvg}&space;\cdot&space;P(2B&space;\cup&space;3B)_{P}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\large&space;P(2B)_{P}=&space;P(2B&space;|&space;2B\cup&space;3B)_{LgAvg}&space;\cdot&space;P(2B&space;\cup&space;3B)_{P}" title="\large P(2B)_{P}= P(2B | 2B\cup 3B)_{LgAvg} \cdot P(2B \cup 3B)_{P}" /></a>

After created models for each player, we then applied **Log5 Method**, a variant of Bayes' formula developed by James, Bill in *Baseball Abstract (1981)* to calculate the outcome distribution for each plate appearance, the formula is represented as follow (e.g. for single hit):

<a href="https://www.codecogs.com/eqnedit.php?latex=\large&space;P(1B)=&space;\frac{(P(1B)_{Batter}\cdot&space;P(1B)_{Pitcher})&space;/&space;P(1B)_{LgAvg}}{(P(1B)_{Batter}\cdot&space;P(1B)_{Pitcher})&space;/&space;P(1B)_{LgAvg}&plus;(1-P(1B)_{Batter})\cdot&space;(1-P(1B)_{Pitcher})&space;/&space;(1-P(1B)_{LgAvg})}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\large&space;P(1B)=&space;\frac{(P(1B)_{Batter}\cdot&space;P(1B)_{Pitcher})&space;/&space;P(1B)_{LgAvg}}{(P(1B)_{Batter}\cdot&space;P(1B)_{Pitcher})&space;/&space;P(1B)_{LgAvg}&plus;(1-P(1B)_{Batter})\cdot&space;(1-P(1B)_{Pitcher})&space;/&space;(1-P(1B)_{LgAvg})}" title="\large P(1B)= \frac{(P(1B)_{Batter}\cdot P(1B)_{Pitcher}) / P(1B)_{LgAvg}}{(P(1B)_{Batter}\cdot P(1B)_{Pitcher}) / P(1B)_{LgAvg}+(1-P(1B)_{Batter})\cdot (1-P(1B)_{Pitcher}) / (1-P(1B)_{LgAvg})}" /></a>

League average probability was calculated with American League stats only because National League lacks of designated hitter would cause average batting performance skewed.
Each plate appearance now has an outcome of either *single, double, triple, home run, or walk*. But there is also a possibility of *error* occurred on the field. Since errors are hardly accountable for the pitcher or the batter's ability, we used conditional probability of an error given the batter doesn't hit or walk, as a constant, which was approximated by the number of errors divided by number of put-outs in the regular season.

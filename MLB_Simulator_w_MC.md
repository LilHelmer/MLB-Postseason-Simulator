MLB Simulator with Monte Carlo
================

``` r
library(rmarkdown)
library(readxl)

## batters stats for each team are collected into different sheets in the same file, the ninth batter in NL is replaced by the league average ("nlninth" sheet)
rangers.bat <- read_excel("Batter_stats.xlsx", sheet = "rangers")
rangers.bat
```

    ## # A tibble: 9 x 17
    ##   order   Pos                           Name   Age     G    PA    AB     R
    ##   <dbl> <chr>                          <chr> <dbl> <dbl> <dbl> <dbl> <dbl>
    ## 1     1    RF     "Shin-Soo Choo*\\choosh01"    33    48   210   178    27
    ## 2     2    CF       "Ian Desmond\\desmoia01"    30   156   677   625   107
    ## 3     3    DH   "Carlos Beltran#\\beltrca01"    39    52   206   193    23
    ## 4     4    3B     "Adrian Beltre\\beltrad01"    37   153   640   583    89
    ## 5     5    2B      "Rougned Odor*\\odorro01"    22   150   632   605    89
    ## 6     6    1B   "Mitch Moreland*\\morelmi01"    30   147   503   460    49
    ## 7     7    SS      "Elvis Andrus\\andruel01"    27   147   568   506    75
    ## 8     8    RF     "Nomar Mazara*\\mazarno01"    21   145   568   516    59
    ## 9     9     C "Robinson Chirinos\\chiriro01"    32    57   170   147    21
    ## # ... with 9 more variables: H <dbl>, `2B` <dbl>, `3B` <dbl>, HR <dbl>,
    ## #   RBI <dbl>, SB <dbl>, CS <dbl>, BB <dbl>, SO <dbl>

``` r
cubs.bat <- read_excel("Batter_stats.xlsx", sheet = "cubs") ## NL team
cubs.bat
```

    ## # A tibble: 9 x 17
    ##   order   Pos                         Name   Age     G    PA    AB     R
    ##   <dbl> <chr>                        <chr> <dbl> <dbl> <dbl> <dbl> <dbl>
    ## 1     1    CF  "Dexter Fowler#\\fowlede01"    30   125   551   456    84
    ## 2     2    3B     "Kris Bryant\\bryankr01"    24   155   699   603   121
    ## 3     3    1B  "Anthony Rizzo*\\rizzoan01"    26   155   676   583    94
    ## 4     4    2B    "Ben Zobrist#\\zobribe01"    35   147   631   523    94
    ## 5     5    SS "Addison Russell\\russead02"    22   151   598   525    67
    ## 6     6    RF  "Jason Heyward*\\heywaja01"    26   142   592   530    61
    ## 7     7    IF      "Javier Baez\\baezja01"    23   142   450   421    50
    ## 8     8     C       "David Ross\\rossda01"    39    67   205   166    24
    ## 9     9     P                  Batting 9th    NA 10722  9179  8148   713
    ## # ... with 9 more variables: H <dbl>, `2B` <dbl>, `3B` <dbl>, HR <dbl>,
    ## #   RBI <dbl>, SB <dbl>, CS <dbl>, BB <dbl>, SO <dbl>

``` r
## pitchers stats for each team are collected into different sheets in the same file, each team has 4 starter pitcher and 5 relief pitchers, 5 rp will then be averaged to form the fifth pitcher of the team
rangers.pit <- read_excel("Pitcher_stats.xlsx", sheet = "rangers")
rangers.pit
```

    ## # A tibble: 9 x 17
    ##   rotation   pos                            Name   Age    IP     G    PA
    ##      <dbl> <chr>                           <chr> <dbl> <dbl> <dbl> <dbl>
    ## 1        1    sp  "Cole\u00a0Hamels*\\hamelco01"    32 200.2    32   848
    ## 2        2    sp "Martin\u00a0Perez*\\perezma02"    25 198.2    33   855
    ## 3        3    sp  "A.J.\u00a0Griffin\\griffaj01"    28 119.0    23   509
    ## 4        4    sp   "Colby\u00a0Lewis\\lewisco01"    36 116.1    19   472
    ## 5        5    rp "Tony\u00a0Barnette\\barneto01"    32  60.1    53   246
    ## 6        5    rp      "Matt\u00a0Bush\\bushma01"    30  61.2    58   243
    ## 7        5    rp "Alex\u00a0Claudio*\\claudal01"    24  51.2    39   217
    ## 8        5    rp "Jake\u00a0Diekman*\\diekmja01"    29  53.0    66   221
    ## 9        5    rp     "Sam\u00a0Dyson\\dysonsa01"    28  70.1    73   285
    ## # ... with 10 more variables: AB <dbl>, R <dbl>, H <dbl>, `2B` <dbl>,
    ## #   `3B` <dbl>, HR <dbl>, SB <dbl>, CS <dbl>, BB <dbl>, SO <dbl>

``` r
er <- 2835/129918  ## error rate is approximated by number of errors divided by number of put-outs in regular season
double <- 8254/(8254+873) ## probability of a double or a triple given a hit is calculated from league average 
triple <- 873/(8254+873)

## create function to transform original dataframe of Batter_stats into multinominal probability model for each players
bat_trans <- function(x) {
    x[["1B"]] <- x[["H"]] - x[["2B"]] - x[["3B"]] - x[["HR"]]
    P.1B <- x[["1B"]]/x[["PA"]]
    P.2B <- x[["2B"]]/x[["PA"]]
    P.3B <- x[["3B"]]/x[["PA"]]
    P.HR <- x[["HR"]]/x[["PA"]]  
    P.BB <- x[["BB"]]/x[["PA"]]
    P.ER <- (1 - (P.1B + P.2B + P.3B + P.HR + P.BB)) * er
    x <- cbind(x[, c(1,3)], P.1B, P.2B, P.3B, P.HR, P.BB, P.ER)
    x <- as.data.frame(x)
}

## create function to transform original dataframe of Batter_stats into multinominal probability model for each players
pitch_trans <- function(x) {
  x[["1B"]] <- x[["H"]] - x[["2B"]] - x[["3B"]] - x[["HR"]]
  P.1B <- x[["1B"]]/x[["PA"]]
  P.2B <- (x[["2B"]]+x[["3B"]])/x[["PA"]] * double 
  P.3B <- (x[["2B"]]+x[["3B"]])/x[["PA"]] * triple
  P.HR <- x[["HR"]]/x[["PA"]]  
  P.BB <- x[["BB"]]/x[["PA"]]
  x <- cbind(x$rotation, P.1B, P.2B, P.3B, P.HR, P.BB)
  x <- rbind(x[c(1:4), ], apply(x[c(5:9), ], 2, mean))
  x <- as.data.frame(x)
}

team_order <- c("rangers", "indians", "redsox", "bluejays", "cubs", "nationals", "dodgers", "giants")
for (i in 1:length(team_order)) {
   bat <- bat_trans(read_excel("Batter_stats.xlsx", sheet = team_order[i]))
   assign(paste0(team_order[i], ".bat"), bat)
   pit <- pitch_trans(read_excel("Pitcher_stats.xlsx", sheet = team_order[i]))
   assign(paste0(team_order[i], ".pit"), pit)
}

rangers.bat
```

    ##   order                         Name       P.1B       P.2B        P.3B
    ## 1     1     Shin-Soo Choo*\\choosh01 0.13809524 0.03333333 0.000000000
    ## 2     2       Ian Desmond\\desmoia01 0.18316100 0.04283604 0.004431315
    ## 3     3   Carlos Beltran#\\beltrca01 0.16990291 0.05825243 0.000000000
    ## 4     4     Adrian Beltre\\beltrad01 0.17343750 0.04843750 0.001562500
    ## 5     5      Rougned Odor*\\odorro01 0.14873418 0.05221519 0.006329114
    ## 6     6   Mitch Moreland*\\morelmi01 0.12723658 0.04174950 0.000000000
    ## 7     7      Elvis Andrus\\andruel01 0.18838028 0.05457746 0.012323944
    ## 8     8     Nomar Mazara*\\mazarno01 0.17781690 0.02288732 0.005281690
    ## 9     9 Robinson Chirinos\\chiriro01 0.07647059 0.06470588 0.000000000
    ##         P.HR       P.BB       P.ER
    ## 1 0.03333333 0.11904762 0.01475546
    ## 2 0.03249631 0.06499261 0.01466582
    ## 3 0.03398058 0.06310680 0.01472419
    ## 4 0.05000000 0.07500000 0.01421804
    ## 5 0.05221519 0.03006329 0.01550290
    ## 6 0.04373757 0.06958250 0.01566112
    ## 7 0.01408451 0.08274648 0.01413785
    ## 8 0.03521127 0.06866197 0.01505988
    ## 9 0.05294118 0.08823529 0.01566010

``` r
rangers.pit
```

    ##   V1      P.1B       P.2B        P.3B       P.HR       P.BB
    ## 1  1 0.1533019 0.03305995 0.003496649 0.02830189 0.09080189
    ## 2  2 0.1649123 0.04865507 0.005146096 0.02105263 0.08888889
    ## 3  3 0.1296660 0.03908781 0.004134196 0.05500982 0.09037328
    ## 4  4 0.1398305 0.03448791 0.003647680 0.04025424 0.05932203
    ## 5  5 0.1554202 0.03315436 0.003506634 0.01551621 0.07341712

``` r
## league average stats are also needed in log5 method calculation, which was collected in Batter_stats 
lgavg <- read_excel("Batter_stats.xlsx", sheet = "lgavg")
lgavg <- data.frame(P.1B = (lgavg[["H"]] - lgavg[["2B"]] - lgavg[["3B"]] - lgavg[["HR"]])/lgavg[["PA"]], 
                    P.2B = lgavg[["2B"]]/lgavg[["PA"]], P.3B = lgavg[["3B"]]/lgavg[["PA"]],
                    P.HR = lgavg[["HR"]]/lgavg[["PA"]], P.BB = lgavg[["BB"]]/lgavg[["PA"]])
```

``` r
## using Log 5 Method to calculate the probability for each plate appearance outcome   
## x = pitcher, y = batter, z = outcome (eg. single, double, triple, etc)
matchup <- function(x, y, z) {
  u <- x[[z]] * y[[z]]/ lgavg[[z]]
  v <- (1-x[[z]])*(1-y[[z]])/(1-lgavg[[z]])
  return(u/(u+v))
}
```

``` r
set.seed(2016)  ## set seed in order to replicate the result in every simulations

##  simulate for the score of the "bat" team facing "pit" team, "no"" is the order of the starting pitchers  
score <- function(pit, bat, no) {
outcome <- c()
inns <- c()
out <- 0
inn <- 1
pts <- numeric(9)

batter_stats <- do.call("rbind", replicate(10, get(paste0(bat, ".bat")), simplify = FALSE))  ## assume each batter has maximum 10 plate appearances
pitcher_stats <- get(paste0(pit, ".pit"))

for (i in 1:(9*10)) {    
        if (inn <= 7) {    ## in first 7 innings, batters would face starting pitcher   
          P.1B <- matchup(pitcher_stats[no, ], batter_stats[i, ], "P.1B")
          P.2B <- matchup(pitcher_stats[no, ], batter_stats[i, ], "P.2B")
          P.3B <- matchup(pitcher_stats[no, ], batter_stats[i, ], "P.3B")
          P.HR <- matchup(pitcher_stats[no, ], batter_stats[i, ], "P.HR")
          P.BB <- matchup(pitcher_stats[no, ], batter_stats[i, ], "P.BB")
          P.ER <- (1 - (P.1B + P.2B + P.3B + P.HR + P.BB)) * er
          P.OUT <- 1 - (P.1B + P.2B + P.3B + P.HR + P.BB + P.ER)
        } else {
          P.1B <- matchup(pitcher_stats[5, ], batter_stats[i, ], "P.1B")
          P.2B <- matchup(pitcher_stats[5, ], batter_stats[i, ], "P.2B")
          P.3B <- matchup(pitcher_stats[5, ], batter_stats[i, ], "P.3B")
          P.HR <- matchup(pitcher_stats[5, ], batter_stats[i, ], "P.HR")
          P.BB <- matchup(pitcher_stats[5, ], batter_stats[i, ], "P.BB")
          P.ER <- (1 - (P.1B + P.2B + P.3B + P.HR + P.BB)) * er
          P.OUT <- 1 - (P.1B + P.2B + P.3B + P.HR + P.BB + P.ER)
        }
          outcome[i] <- sample(c("1B", "2B", "3B", "HR", "BB", "Out", "ER"), 1, prob = c(P.1B, P.2B, P.3B, P.HR, P.BB, P.OUT, P.ER))
          inns[i] <- inn 
    
          if (outcome[i] == "Out") {out <- out + 1}
          if (out == 3) {inn <- inn + 1; out <- 0}    ## proceed to next inning after 3 outs
          if (inn == 10) {break}     ## game breaks after 9th innings (27 outs)
} 

outcome_inns <- rbind(outcome, inns) 
## to count the score with base counter logic (in README.md)
for (k in 1:9) {
  x <- outcome_inns[1, inns == k]
  outcome_no <- x[-which(x == "Out")]
  bases <- numeric(length(outcome_no))
  if (length(bases) == 0) {
    next
  } else {
  for (j in 1:length(outcome_no)) {
      if (outcome_no[j] %in% c("1B", "ER")) {
        bases[1:j] <- bases[1:j] + 1
      } else if (outcome_no[j] == "2B") {
        bases[1:j] <- bases[1:j] + 2
      } else if (outcome_no[j] == "3B") {
        bases[1:j] <- bases[1:j] + 3
      } else if (outcome_no[j] == "HR") {
        bases[1:j] <- bases[1:j] + 4
      } else {
          if ((j == 2 && bases[j-1] == 1) || (j == 3 && bases[j-1] == 1 && bases[j-2] == 2) || (j >= 4 && bases[j-1] == 1 && bases[j-2] == 2 && bases[j-3] == 3)) {
            bases[1:j] <- bases[1:j] + 1
          } else if (j == 3 && bases[j-1] == 1 && bases[j-2] == 3) {
            bases[j-1:j] <- bases[j-1:j] + 1 
          } else {bases[j] <- 1}
      } 
  }
  }  
  pts[k] <- sum(bases >= 4)
}
  return(sum(pts))

}

## simulate the outcome for one game: compare the scores of two teams and output the winner 
game_outcome <- function(team1, team2, no) {
  repeat {
    score2 <- score(team1, team2, no)
    score1 <- score(team2, team1, no)
    if (score1 > score2) {y <- team1} else if (score2 > score1) {y <- team2} else {y <- "even"}
    if (y != "even") {break}  ## repeat simulation if it's even
  }
  return(y)
}

## simulate the outcome for one series, "games"" is the number of games in the series (bo5 or bo7) 
series_outcome <- function(team1, team2, games) {
  u <- 0
  v <- 0
  if (games == 5) {
    for (i in c(1:4,1)) {
      winner <- game_outcome(team1, team2, i)
      if (winner == team1) {u <- u + 1} else if (winner == team2) {v <- v + 1}
      if (u >= 3 || v >= 3) {break}
    }
  } else if (games == 7) {
    for (i in c(1:4,1:3)) {
      winner <- game_outcome(team1, team2, i)
      if (winner == team1) {u <- u + 1} else if (winner == team2) {v <- v + 1}
      if (u >= 4 || v >= 4) {break}
    }
  }
  if (u > v) {return(team1)} else {return(team2)} 
}
```

``` r
M <- 200  ## simulate 200 times for each series

#########################    LDS (BO5)    ###############################
rangers.bluejays <- character(M)
indians.redsox <- character(M)
dodgers.nationals <- character(M)
giants.cubs <- character(M)


for (j in 1:M) {
  rangers.bluejays[j] <- series_outcome("rangers", "bluejays", 5)
  indians.redsox[j] <- series_outcome("indians", "redsox", 5)
  dodgers.nationals[j] <- series_outcome("dodgers", "nationals", 5)
  giants.cubs[j] <- series_outcome("giants", "cubs", 5)
}

LDS <- data.frame(team = c("rangers", "bluejays", "indians", "redsox", "dodgers", "nationals", "giants", "cubs"),
                  prob.lds = c(sum(rangers.bluejays == "rangers")/M, sum(rangers.bluejays == "bluejays")/M, 
                               sum(indians.redsox == "indians")/M, sum(indians.redsox == "redsox")/M,
                               sum(dodgers.nationals == "dodgers")/M, sum(dodgers.nationals == "nationals")/M,
                               sum(giants.cubs == "giants")/M, sum(giants.cubs == "cubs")/M))
LDS
```

    ##        team prob.lds
    ## 1   rangers    0.460
    ## 2  bluejays    0.540
    ## 3   indians    0.330
    ## 4    redsox    0.670
    ## 5   dodgers    0.375
    ## 6 nationals    0.625
    ## 7    giants    0.250
    ## 8      cubs    0.750

``` r
#########################    team matchups within each league (BO7)    ###############################
rangers.bluejays.7 <- character(M)
rangers.indians.7 <- character(M)
rangers.redsox.7 <- character(M)
bluejays.indians.7 <- character(M)
bluejays.redsox.7 <- character(M)
indians.redsox.7 <- character(M)

dodgers.nationals.7 <- character(M)
dodgers.giants.7 <- character(M)
dodgers.cubs.7 <- character(M)
nationals.giants.7 <- character(M)
nationals.cubs.7 <- character(M)
giants.cubs.7 <- character(M)


for (j in 1:M) {
  rangers.bluejays.7[j] <- series_outcome("rangers", "bluejays", 7)
  rangers.indians.7[j] <- series_outcome("rangers", "indians", 7)
  rangers.redsox.7[j] <- series_outcome("rangers", "redsox", 7)
  bluejays.indians.7[j] <- series_outcome("bluejays", "indians", 7)
  bluejays.redsox.7[j] <- series_outcome("bluejays", "redsox", 7)
  indians.redsox.7[j] <- series_outcome("indians", "redsox", 7)
 
  dodgers.nationals.7[j] <- series_outcome("dodgers", "nationals", 7)
  dodgers.giants.7[j] <- series_outcome("dodgers", "giants", 7)
  dodgers.cubs.7[j] <- series_outcome("dodgers", "cubs", 7)
  nationals.giants.7[j] <- series_outcome("nationals", "giants", 7)
  nationals.cubs.7[j] <- series_outcome("nationals", "cubs", 7)
  giants.cubs.7[j] <- series_outcome("giants", "cubs", 7)
}


al_bo7 <- data.frame(losing_team = c("rangers", "bluejays", "indians", "redsox"),
                     rangers = c(NA, sum(rangers.bluejays.7=="rangers"), sum(rangers.indians.7=="rangers"), sum(rangers.redsox.7=="rangers"))/M,
                     bluejays = c(sum(rangers.bluejays.7=="bluejays"), NA, sum(bluejays.indians.7=="bluejays"), sum(bluejays.redsox.7=="bluejays"))/M, 
                     indians = c(sum(rangers.indians.7=="indians"), sum(bluejays.indians.7=="indians"), NA, sum(indians.redsox.7=="indians"))/M,
                     redsox = c(sum(rangers.redsox.7=="redsox"), sum(bluejays.redsox.7=="redsox"), sum(indians.redsox.7=="redsox"), NA)/M
                    )
al_bo7 
```

    ##   losing_team rangers bluejays indians redsox
    ## 1     rangers      NA    0.625   0.495   0.80
    ## 2    bluejays   0.375       NA   0.435   0.71
    ## 3     indians   0.505    0.565      NA   0.76
    ## 4      redsox   0.200    0.290   0.240     NA

``` r
nl_bo7 <- data.frame(losing_team = c("dodgers", "nationals", "giants", "cubs"),
                     dodgers = c(NA, sum(dodgers.nationals.7=="dodgers"), sum(dodgers.giants.7=="dodgers"), sum(dodgers.cubs.7=="dodgers"))/M,
                     nationals = c(sum(dodgers.nationals.7=="nationals"), NA, sum(nationals.giants.7=="nationals"), sum(nationals.cubs.7=="nationals"))/M,
                     giants = c(sum(dodgers.giants.7=="giants"), sum(nationals.giants.7=="giants"), NA, sum(giants.cubs.7=="giants"))/M,
                     cubs = c(sum(dodgers.cubs.7=="cubs"), sum(nationals.cubs.7=="cubs"), sum(giants.cubs.7=="cubs"), NA)/M
                     )
nl_bo7
```

    ##   losing_team dodgers nationals giants cubs
    ## 1     dodgers      NA     0.485  0.430 0.54
    ## 2   nationals   0.515        NA  0.365 0.69
    ## 3      giants   0.570     0.635     NA 0.76
    ## 4        cubs   0.460     0.310  0.240   NA

``` r
#########################    team matchups between each league (BO7)    ###############################
rangers.dodgers.7 <- character(M)
rangers.nationals.7 <- character(M)
rangers.giants.7 <- character(M)
rangers.cubs.7 <- character(M)
bluejays.dodgers.7 <- character(M)
bluejays.nationals.7 <- character(M)
bluejays.giants.7 <- character(M)
bluejays.cubs.7 <- character(M)
indians.dodgers.7 <- character(M)
indians.nationals.7 <- character(M)
indians.giants.7 <- character(M)
indians.cubs.7 <- character(M)
redsox.dodgers.7 <- character(M)
redsox.nationals.7 <- character(M)
redsox.giants.7 <- character(M)
redsox.cubs.7 <- character(M)

for (j in 1:M) {
  rangers.dodgers.7[j] <- series_outcome("rangers", "dodgers", 7)
  rangers.nationals.7[j] <- series_outcome("rangers", "nationals", 7)
  rangers.giants.7[j] <- series_outcome("rangers", "giants", 7)
  rangers.cubs.7[j] <- series_outcome("rangers", "cubs", 7)
  bluejays.dodgers.7[j] <- series_outcome("bluejays", "dodgers", 7)
  bluejays.nationals.7[j] <- series_outcome("bluejays", "nationals", 7)
  bluejays.giants.7[j] <- series_outcome("bluejays", "giants", 7)
  bluejays.cubs.7[j] <- series_outcome("bluejays", "cubs", 7)
  indians.dodgers.7[j] <- series_outcome("indians", "dodgers", 7)
  indians.nationals.7[j] <- series_outcome("indians", "nationals", 7)
  indians.giants.7[j] <- series_outcome("indians", "giants", 7)
  indians.cubs.7[j] <- series_outcome("indians", "cubs", 7)
  redsox.dodgers.7[j] <- series_outcome("redsox", "dodgers", 7)
  redsox.nationals.7[j] <- series_outcome("redsox", "nationals", 7)
  redsox.giants.7[j] <- series_outcome("redsox", "giants", 7)
  redsox.cubs.7[j] <- series_outcome("redsox", "cubs", 7)
}

all_bo7 <- data.frame(losing_team = c("dodgers", "nationals", "giants", "cubs"),
                      rangers = c(sum(rangers.dodgers.7=="rangers"), sum(rangers.nationals.7=="rangers"), sum(rangers.giants.7=="rangers"), sum(rangers.cubs.7=="rangers"))/M, 
                      bluejays = c(sum(bluejays.dodgers.7=="bluejays"), sum(bluejays.nationals.7=="bluejays"), sum(bluejays.giants.7=="bluejays"), sum(bluejays.cubs.7=="bluejays"))/M,
                      indians = c(sum(indians.dodgers.7=="indians"), sum(indians.nationals.7=="indians"), sum(indians.giants.7=="indians"), sum(indians.cubs.7=="indians"))/M,
                      redsox = c(sum(redsox.dodgers.7=="redsox"), sum(redsox.nationals.7=="redsox"), sum(redsox.giants.7=="redsox"), sum(redsox.cubs.7=="redsox"))/M
                      )
all_bo7
```

    ##   losing_team rangers bluejays indians redsox
    ## 1     dodgers   0.365    0.405   0.520  0.675
    ## 2   nationals   0.315    0.420   0.340  0.610
    ## 3      giants   0.430    0.570   0.580  0.815
    ## 4        cubs   0.245    0.315   0.285  0.520

``` r
prob.lcs = c(LDS$prob.lds[1] * (LDS$prob.lds[3]*al_bo7$rangers[3] + LDS$prob.lds[4]*al_bo7$rangers[4]),     ## rangers wins LCS
             LDS$prob.lds[2] * (LDS$prob.lds[3]*al_bo7$bluejays[3] + LDS$prob.lds[4]*al_bo7$bluejays[4]),   ## bluejays wins LCS
             LDS$prob.lds[3] * (LDS$prob.lds[1]*al_bo7$indians[1] + LDS$prob.lds[2]*al_bo7$indians[2]),     ## indians wins LCS
             LDS$prob.lds[4] * (LDS$prob.lds[1]*al_bo7$redsox[1] + LDS$prob.lds[2]*al_bo7$redsox[2]),       ## redsox wins LCS
             
             LDS$prob.lds[5] * (LDS$prob.lds[7]*nl_bo7$dodgers[3] + LDS$prob.lds[8]*nl_bo7$dodgers[4]),     
             LDS$prob.lds[6] * (LDS$prob.lds[7]*nl_bo7$nationals[3] + LDS$prob.lds[8]*nl_bo7$nationals[4]),   
             LDS$prob.lds[7] * (LDS$prob.lds[5]*nl_bo7$giants[1] + LDS$prob.lds[6]*nl_bo7$giants[2]),     
             LDS$prob.lds[8] * (LDS$prob.lds[5]*nl_bo7$cubs[1] + LDS$prob.lds[6]*nl_bo7$cubs[2])
             )

prob.ws = numeric(8)
for (i in 1:8) {
  if (i <= 4) {
    prob.ws[i] = prob.lcs[i] * (prob.lcs[5]*all_bo7[1, i+1] + prob.lcs[6]*all_bo7[2, i+1] + prob.lcs[7]*all_bo7[3, i+1] + prob.lcs[8]*all_bo7[4, i+1])
  } else {
    prob.ws[i] = prob.lcs[i] * (prob.lcs[1]*(1-all_bo7[i-4, 2]) + prob.lcs[2]*(1-all_bo7[i-4, 3]) + prob.lcs[3]*(1-all_bo7[i-4, 4]) + prob.lcs[4]*(1-all_bo7[i-4, 5]))
  }
}

final_result <- cbind(LDS, prob.lcs, prob.ws)
final_result
```

    ##        team prob.lds   prob.lcs    prob.ws
    ## 1   rangers    0.460 0.13829900 0.04177505
    ## 2  bluejays    0.540 0.20560500 0.07853115
    ## 3   indians    0.330 0.15265800 0.05650278
    ## 4    redsox    0.670 0.50343800 0.30158965
    ## 5   dodgers    0.375 0.18281250 0.08172597
    ## 6 nationals    0.625 0.24453125 0.12497523
    ## 7    giants    0.250 0.09734375 0.03158736
    ## 8      cubs    0.750 0.47531250 0.28331281

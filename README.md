# League of Legends Outcome Predictor After 10 Minutes

## Framing the Problem:
The League of Legends dataset loaded above give insight on professional games from the 2022 season. It specifies information and statistics for each player for each specific game, including bans, kills, deaths, damage taken per minute, wards placed, etc. As a beginner to the game, I wonder if there are aspects of the game that could be predicted. The specific question that I will be focusing on predicting will be -- which team will win or lose a game after the first 10 minutes have played out? 

I will be performing binary classification to predict whether or not a given team or not wins using data only found at the 10 minute mark. This data includes columns such as killsat10, deathsat10, assistsat10 in addition to categorical columns such as the 5 champions chosen. The metric I will be using to evaluate the model I am building is going to be accuracy. I will be using this metric over other ones because it is a good gauge on if my model's predictions are accurate or not because it tells me the percentage of data points classified correctly / total data pionts. This will be more beneficial to me than using other metrics that calculate precision, recall, etc. 

I am using the League of Legends Competitive Matches 2022 dataset (https://oracleselixir.com/tools/downloads). This original dataset includes 149232 rows and 123 columns, but I filtered it down to only include the rows for team statistics. Additionally, I removed the columns that weren't relevant to my question. I only kept the columns involving stats at the 10 minute mark, or general game information that is known at the start of the game. My final dataset has 24987 rows and 40 rows.

### Data Cleaning:
This League of Legends Competitive Matches 2022 dataset had a lot of missing data, and as such, had to be filled in accordingly. For columns like “url” and “split”, I filled in the NaN values as “unknown” because these aren’t fields that could be filled in using other values in that column. For columns such as ban1, ban2, ..., ban5, I filled them in with "None". For other numerical columns such as killsat10, assistsat10, deathsat10, etc., I imputed the NaN values with the median for each respective column. Finally, for columns “result” and “playoff”, I changed the type of the variables from 1/0 to True/False. The first few rows of the new cleaned dataset with a few chosen columns can be seen here:

The first few rows of the new cleaned dataset with a few chosen columns can be seen here:

| gameid                | datacompleteness   |   killsat10 |   assistsat10 |   deathsat10 |
|:----------------------|:-------------------|------------:|--------------:|-------------:|
| ESPORTSTMNT01_2690210 | complete           |           3 |             5 |            0 |
| ESPORTSTMNT01_2690210 | complete           |           0 |             0 |            3 |
| ESPORTSTMNT01_2690219 | complete           |           1 |             1 |            3 |
| ESPORTSTMNT01_2690219 | complete           |           3 |             3 |            1 |
| 8401-8401_game_1      | partial            |           2 |             2 |            2 |


## Baseline Model:
For my baseline model, I transformed quantitative columns killsat10, assistsat10, and deathsat10 using sklearn's StandardScaler. For my classifier, I used the DecisionTreeClassifier with an arbitrary max_depth of 2. I split the data set into training and testing sets using sklearn's train_test_split. I put the column transformation and my classifier into a pipeline, and then fitted it using my training set. I then computed the accuracy score on the testing set, and it resulted in a value of 0.5812. Evidently, this isn't that good, as my model is wrong a little less than half of the time. 

## Final Model:
For my final model, I transformed a few more quantitative columns (in addition to the ones in my baseline model) including goldat10, xpat10, csat10, golddiffat10, etc. Additionally, I encoded nominal columns champion1, champion2, ..., champion5 using sklearn's OneHotEncoder. I still used the same classicfier in the DecisionTreeClassifier. However, for this step, I used GridCV to calculate optimal hyperparameters to improve the scoring accuracy of my model. Out of all possible combinations of max_depth, min_samples_split, and criterion, it found that the optimal choices were: max_depth = 5, min_samples_split = 100, and criterion = entropy. I inputted these parameters when instantiating my DecisionTreeClassifier, and then put it in the pipeline. The accuracy value for my final model on the testing set after including more features and finetuning the hyperparameters was 0.67 -- this is ~0.10 of an increase from my baseline model!

## Fairness Analysis:
To test the fairness  of my final model, I decided to split my dataset into two groups -- one where the game's gamelength is greater than or equal to the average and one where the game's gamelength is less than the average. I will perform a permutation test to decide whether or not the precision calculated is fair for both groups. I will use a significance level cutoff of 0.05. 

- Null hypothesis is that our model is fair. Its precision for games of length less than the average game length and for games of length greater than the average game length are roughly the same. Any difference is due to random chance. 
- Alternative hypothesis is our model is unfair. Its precision for games with length greater than average is lower than the games with length lower than average. 

I will run 500 iterations, where for each iteration, I will shuffle the "gamelength" columm randomly. I will then calculate the test statistic of the difference of precision from the games with average game length and >= average game length. 

The p-value for one iteration of this permutation test was 0.006 which is less than the significance level of 0.06. As a result, we reject the null, and that the precision for games with length greater than average is lower than the games with length lower than average. In competitive matches at least, my model calculates the outcome better for short games. This logically makes sense though, as if a game is shorter, then a higher proportion of the game has already played out at the 10 minute mark.



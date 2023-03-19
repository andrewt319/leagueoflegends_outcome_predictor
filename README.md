# League of Legends Outcome Predictor After 10 Minutes of Gameplay

## Framing the Problem:
The League of Legends dataset loaded above give insight on professional games from the 2022 season. It specifies information and statistics for each player for each specific game, including bans, kills, deaths, damage taken per minute, wards placed, etc. As a beginner to the game, I wonder if there are aspects of the game that could be predicted. The specific question that I will be focusing on predicting will be -- which team will win or lose a game after the first 10 minutes have played out? 

I will be performing binary classification to predict whether or not a given team or not wins using data only found at the 10 minute mark. This data includes columns such as killsat10, deathsat10, assistsat10 in addition to categorical columns such as the 5 champions chosen. The metric I will be using to evaluate the model I am building is going to be accuracy. I will be using this metric over other ones because it is a good gauge on if my model's predictions are accurate or not because it tells me the percentage of data points classified correctly / total data pionts. This will be more beneficial to me than using other metrics that calculate precision, recall, etc. 

I am using the League of Legends Competitive Matches 2022 dataset (https://oracleselixir.com/tools/downloads). This original dataset includes 149232 rows and 123 columns, but I filtered it down to only include the rows for team statistics. Additionally, I removed the columns that weren't relevant to my question. I only kept the columns involving stats at the 10 minute mark, or general game information that is known at the start of the game. My final dataset has 24987 rows and 40 columns.

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
For my baseline model, I transformed quantitative columns killsat10, assistsat10, and deathsat10 using sklearn's StandardScaler. For my classifier, I used the DecisionTreeClassifier with an arbitrary max_depth of 2. I split the data set into training and testing sets using sklearn's train_test_split. I put the column transformation and my classifier into a pipeline, and then fitted it using my training set. I then computed the accuracy score on the testing set, and it resulted in a value of 0.5812. This isn't that bad, considering we can predict the correct outcome a little more than half of the time. But, it also isn't that great either.

## Final Model:
For my final model, I transformed a few more quantitative columns (in addition to the ones in my baseline model) including goldat10, xpat10, csat10, golddiffat10, etc. Additionally, I encoded nominal columns champion1, champion2, ..., champion5 using sklearn's OneHotEncoder. I still used the same classifier in the DecisionTreeClassifier. These additional features I added help to cover more aspects of the game. There's more to a League of Legends game than just kills, deaths, and assists -- there's aspects such as gold, xp, and cs that help to describe and predict future parts of the game. For example, maybe a certain team's game plan is to farm up gold so mid to late game they can purchase more expensive items in the shop. I also included champions as features in my model because there's a possibility that a specific combination of champions is optimal in getting more kills, more assists, more gold, etc. It's important to take these aspects of the game into consideration, which is why I included them as features in my modeling. 

I used GridCV to calculate optimal hyperparameters to improve the scoring accuracy of my model. Out of all possible combinations of max_depth, min_samples_split, and criterion, it found that the optimal choices were: max_depth = 3, min_samples_split = 2, and criterion = gini. I inputted these parameters when instantiating my DecisionTreeClassifier, and then put it in the pipeline. The accuracy value for my final model on the testing set after including more features and finetuning the hyperparameters was 0.67. This is about 0.10 of an increase in accuracy from my baseline model!

Here's a confusion matrix that helps to represent my final model's performance.
<iframe src="assets/confusion_matrix.png" width=600 height=600 frameBorder=0></iframe>


## Fairness Analysis:
To test the fairness  of my final model, I decided to split my dataset into two groups -- one where the game's gamelength is greater than or equal to the average and one where the game's gamelength is less than the average. I will perform a permutation test to decide whether or not the precision calculated is fair for both groups. I will use a significance level cutoff of 0.05. 

- Null hypothesis is that our model is fair. Its precision for games of length less than the average game length and for games of length greater than the average game length are roughly the same. Any difference is due to random chance. 
- Alternative hypothesis is our model is unfair. Its precision for games with length greater than average is lower than the games with length lower than average. 

I will run 500 iterations, where for each iteration, I will shuffle the "gamelength" columm randomly. I will then calculate the test statistic of the difference of precision from the games with average game length and >= average game length. 

The p-value for one iteration of this permutation test was 0.006 which is less than the significance level of 0.05. As a result, we reject the null, and that the precision for games with length greater than average is lower than the games with length lower than average. In competitive matches at least, my model calculates the outcome better for short games. This logically makes sense though, as if a game is shorter, then a higher proportion of the game has already played out at the 10 minute mark.


## Conclusion:
The model I built is able to predict whether or not a specific team will win or not after 10 minutes of gameplay with an accuracy of 67%. Although this isn't that high, keep in mind that this model uses professional matches as training data. The professional league players will do a good job at not letting a bad start determine the outcome of a game -- moreso than just casual gamers. Although this is purely speculative, I would think that if I were to perform this exact same modeling and training on a dataset amongst casual games, that this accuracy would be higher. However, amongst its current state in professional games after 10 minutes of gameplay, the model that I created will be able to accuractely predict the outcome of a game around 2/3 of the time. 
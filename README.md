## Entity Resolution
### Author: Zhifu Xiao
### Date: 03/25/2017

## Introduction

In this project, the input data are two datasets from Amazon and RottenTomato describing some features of movies, and the task is to examine whether two entries from each dataset are the same or not, which is also known as the entity resolution problem. After doing data preprocessing, generate training and test set and apply classifiers, the program could output the predicted value in the test set and holdout set. Overall, the program gives an F1 score of 0.9625 on the test set. In the next several chapters I will describe the detailed processes in each step.

### Data pre-processing

In the data preprocessing step, I first took a look at the datasets from Amazon and RottenTomato and determined which features are useful. From the dataset, we could see that both of them contains the movie length, director name and stars' names. In Amazon dataset, it also contains the price of the movies, while in RottenTomato dataset, it contains the screening date, rating, and some comments. I decided to only use the common features as the predictors. After dropping the redundant columns, I found out that in Amazon dataset, the stars contains more than one actor/actress, and I split them out and make separate columns. Also, in Amazon dataset, there some entries that the movie time and stars are swapped. I swapped them back and corrected it. In RottenTomato dataset, I found out that there are some missing entries with no movie time, and I filled them with None. Moreover, both of them have different notations in movie time, and I used minutes to unify them and make it clear. After the data pre-processing step, the datasets are now cleaned, with id, movie time in minutes, director name and stars' names.

### Generate training and test set

In the next step, since the training and test file have the movie id from the two datasets, I first extracted the right entries from two datasets, then I create the time difference between two movie times, which is the absolute difference between two movie times. Then I used a package called **[fuzzywuzzy](https://github.com/seatgeek/fuzzywuzzy)** to apply fuzzy string match and generate matching scores on the director name and stars' names. Note that matching director name is easy since there is one and only one director from each dataset. However, the stars are different in two datasets: Amazon dataset have one or two stars, while RottenTomato dataset has at most six stars. Here I first create a matching score based on the Amazon dataset: if there are two stars, each of them will find their best matching among RottenTomato dataset and receive two scores; if there is only one star, it will find their best matching among RottenTomato dataset and with the median score among the dataset. After this process, there are two scores, and to create a single value, I used a formula to calculate the single score of the stars' similarity: Call the set of two scores $S$, then we have

$$score = \frac{ \big( \frac{S_{max}} {100} \big) ^ 2 + \frac{S_{min}} {100} } {2} \times 100$$

This notation ensures that if two start both matches, then the output score is 100; if one of them perfectly matches and the other one mostly matches, the output score is still very high (take 100, 90 as input, then the output is 95. If only one of the scores is perfectly matched and the other one is not, then the output is mediocre (take 100, 20 as input, then the output is 60). If both of them do not match, then the output is low ( take 50, 50 as input, then the output is 37.5. If one of them is missing, then it will use the median value to fill the gap and generate the result. Overall, the score is moving toward the one that has perfectly match, no matter one or two; it is less favor to two mediocre matches and penalizes by the square. After generating the star score, it will then bind those columns together to make a formal training set and test set.

### Apply classifiers

After generating training and test set, I used Robust scalar and AdaBoosting classifier to fit the data into the model and make the prediction. After making prediction on the test dataset and holdout dataset, I then output the results into .csv files with header gold.

### Questions

**1. Describe your entity resolution technique, as well as its precision, recall, and F1 score.**

** Answer: ** I explained the technique in the several previous sections, and the precision, recall, and F1 score are listed below.

| Label | Precision | Recall | F1 Score | Support |
|-------|-----------|--------|----------|---------|
| 0    | 98.28%   | 100%  | 0.9913   | 57    |
| 1    | 100%     | 83%  | 0.9091   | 6     |

From the results, we could see that precision, recall and F1 Score are high and I believe that the result is convincing.

** 2. What were the most important features that powered your technique?**

** Answer: **  From the results, we could say that there are only three features, and all of them are important in the model, and the first two are probably more important: there are some examples that all of the three features are matched, and most of them are the same movie. However, there are more examples that the two out of three mostly match: movie length and director name, and by AdaBoosting technique, the model will favor the ordinary examples and enhanced by iterations.

** 3. How did you avoid pairwise comparison of all movies across both datasets?** 

** Answer: ** In this assignment, I didn't use the original datasets as the training datasets; on the other hand, I extracted the features in the two original datasets and generated the training dataset by the index number in the file. By doing that, I avoided using the 5000 rows dataset and make the pairwise comparison of all movies across both datasets. Instead, the new training dataset only has the same rows as the training index file, which is around 250 rows. The computational complexity and running time are greatly reduced and could generate the predicted result much faster.

## Reference and note

> FuzzyWuzzy by seatgeek, retrived through pip install fuzzywuzzy at [https://github.com/seatgeek/fuzzywuzzy](https://github.com/seatgeek/fuzzywuzzy)

> The original sourcecode is in a seprate .ipynb file called sourcecode.ipynb

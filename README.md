# SUUMO_Central_TOKYO

Let's explore second-hand apartments in Central Tokyo!



## 0, Abstruct
Using data scraped from SUUMO, I analyzed second-hand apartments in Central Tokyo to identify promising investment opportunities. After performing EDA, I applied regression analysis, clustering, and frequent pattern mining. I found that undervalued apartments tend to have older buildings, simple layouts, medium sizes, and are located near stations but in less popular areas. These characteristics may indicate good investment potential.


## 1, Introduction
Currently, in Japan, many people are moving to Tokyo, especially central Tokyo because it offers a lot of places to enjoy and job oppourtunities, with higher salaries compared to other prefectures. Additionally, many forenighers are living in or visting Tokyo due to the weakening yen. As a result, the price of apartment in Tokyo has been increasing. Living in the central Tokyo is a dream for most people, which means there is a high demand for apartments there. However, since brand-new apartments are quite expensive, when it comes to purchasing, second-hand apartments are often a more realistic choice than new ones. Therefore, this analysis focuses on second-hand apartments in central Tokyo. 



## 2, Questions
The main purpose of this analysis is:

- Which types of second-hand apartments are ideal for investment.

To answer the main question, I broke it down into three sub-questions:

1.  Analysis of the most valueable areas and popular types of apartments
2.  Analysis of factors that affect apartment prices significantly
3.  Analysis of undervalued apartments

By answering these questions, I can find the best types of second-hand apartments for investiment.



## 3, Who Utilizes the Analysis
- Individuals looking to invest apartments in central Tokyo
- People who are searching for apartments to live in
- Property owners who want to sell their apartments but are unsure of the market benchmark
- Real estate agents seeking insights into market trends



## 4, The Way of Analysis

### 4-1. The dataset
For this analysis, I collected data from SUUMO using webscraping. [url: https://suumo.jp/kanto/]


### 4-2. Cleaning and Preprocessing method
I conducted data cleaning and preprocessing:

For each column, I performed preprocessing below:

- `Price` column: Remove Kanji
- `Address` column: Split it into three columns; `prefecture`, `city`, `town`
- `station_distance` column: Split it into four columns; `line`, `station`, `way`, `walking time`
- `area` column: Remove unit(m^2) and the brackets
- `floor_plan` column: Split it into four columns; `room`, `L`, `D`, `K`, `S`. Also, removed the brackets
- `balcony` column: Remove unit(m^2)
- `year` column: Split it into two columns; `date_year`, `date_month` and create new column `age`

I also removed rows with inconsistent or invalid values.

Additionally, I used population data in central tokyo from "Statistics of Tokyo" [url: https://www.toukei.metro.tokyo.lg.jp/juukiy/jy-index.htm], and combined it to the dataframe.

Lastly, I made a new column: `popularity` by dividing `count` by `population`.

### 4-3. Exploratory Data Analysis (EDA)

I visualized data using the preprocessed the data frame. What I found by visualizing data is as follows:

- The histogram for the Age of building shows that there are three peaks at around 3 years, 15 years and 45 years.

- "港区" is the most popular city and "文京区" is the worst city

- The apartment built in Winte season has the highest price/m^2 among the all seasons

- "港区" is the most expensive city and "文京区" is the worst.

- The number of apartment in "千代田区" is the lowest

- If the aparment has the S or not is not a big deal.

- "虎ノ門ヒルズ" is the most expensive station, following "六本木一丁目"，"溜池山王", and "御茶ノ水"

- "東急東横線" is the most expensive station, following "東京メトロ南北線"

- The higher `age` is, the cheaper the price/m2 is 

- Walking time does not affect the price/m2 that much, except in "港区"


### 4-4. Machine Learning Models
#### 4-4-1. Regression
To estimate apartment prices, I applied Multiple Linear Regression (MLR), which is a simple but interpretable model suitable for understanding how each feature contributes to the price.

Before applying MLR, I removed outliers and drop `town` column. Also, for categorical varialbes, I performed one-hot encoding. For `line` and `station` columns, I picked up some values as they have over 50 columns. Additionally, I apply standarization and log transformation.

I evaluated the model using R-squared, and got feature coefficients to understand which features increase or decrease the apartment price.


#### 4-4-2. Clustering
I applied clustering to find hidden patterns in the dataset. I used K-Mean clustering, which groups data into k clusters based on feature similarity.

Before applying K-Mean, I perfored frequency encoding for `city`, `line`, and `station` columns. Also, I applied standardization. To decide the number of clusters (k), I used the Elbow method, and I found that the k=6 is the best number of clusters.

Then, outputted the table and visualzed the table with the spider plot.


#### 4-4-3. Frequent Pattern Mining
I applied frequent pattern mining to find out which combinations of features appear frequently in the dataset. This technique helps to discover patterns that are common among undervalued apartments, which can be useful for investment decisions.

Before applying frequent pattern mining, I converted numerical columns into categorical values. For example, I divided the `walking time` column into 5 bins and assigned category labels. I applied the similar binning process to the `area [m2]`, `age`, and `residual` columns.

After converting the numerical features into categorical ones, I applied one-hot encoding to all features to prepare the data for the Apriori algorithm. Additionally, I selected features that are important for investment as antecedents based on EDA and the coefficients from the regression model, and used undervalued apartments (positive residuals) as consequents.

I set the minimum support for the Apriori algorithm to 0.05, and the minimum confidence threshold for generating rules to 0.5. Then, I performed frequent pattern mining to find out what kinds of apartments tend to be undervalued by checking three metrics; support, confidence and lift.

Each Metric represents:

- Support x%:  appears in approximately x% of all properties

- Confidence x%: x% chance that a property is underpriced when this condition is met

- Lift x%: The probability of a property being underpriced is x times higher compared to random chance



## 5. Results
Using the machine leraning models, I got results below:

#### For regression,
To checke how the regression model is accurate, I used the R-squared, and the result is 0.9. So, I can say that the model is quite accurate.
Then, I computed effecient elements and Top 5 are below:

- age : -0.625
- room : 0.151
- walking time : -0.127
- city_港 : 0.124
- popularity : 0.118

#### For Clustering,
I found that 6 cluster is the best for this dataset, and each cluster represents below:

- Cluster 0: Dense or older area

 => High (`population`, `age`) | Low (`popularity`, `residual`, `price/m2`)

- Cluster 1: top-tier and high-demand area

=> High (`price/m2`, `city`, `popularity`) | Low (`age`)

- Cluster 2: Far but large apartment

=> High (`station`, `walking time`, `balcony`) | Low (`popularity`, `price/m2`)

- Cluster 3: low-demand and narrow aparment

=>  High (NA) | Low (`popularity`, `price/m2`)

- Cluster 4: Poor layout

 => High (NA) | Low (`station`, `city`, `line`, `residual`, `room`, `L`, `D`, `K`, `S`)
 
- Cluster 5: Expensive but not popular

=> High (`city`, `price/m2`, 'popularity') | Low (`population`)

However, the silhouette score was only 0.19, which suggests the clusters may not be well-separated.

#### For Frequent pattern mining,
Using the results from the regression model and EDA, I defined popular elements for investment. After that, I performed frequent pattern mining and looked for undervalued apartment which include popular elements I defined before.

Then, I found the undervalued apartment tend to have elements below:
- area [m2]_medium, age_old
- walking time_close, age_old
- room_2, age_old



## 6. Conclusion

#### Question 1.  What is the most valueable areas and popular types of apartments? 
Using EDA, I can answer the question 1,

- The most valueable city is "港区"
- The most expensive station is "虎ノ門ヒルズ"
- The most expensive line is "東急東横線"
- The range of age is between 0 and 10
- The number of room is 2 or 3
- Has Living, Dining and Kitchen

Additionally, from the clustering method, we can know that Cluster 1 and 5 are popular because those popularity are quite high. These clusters tend to include apartments located in major cities with a multiple room and L, D, K, and S.


#### Question 2: What factors affect apartment prices significantly?
Using the result from regression model, I can answer the question 2,

- The age of building affected negatively to price/m2 the most significantly.
- The large number of room increase the price/m2.
- The higher walking time affected decrease the price/m2.


#### Question 3: Which types of apartment is underpriced?
Using the result from frequent pattern mining, I can answer question 3,

- area [m2]_medium, age_old => support: 5%, confidence: 61%, lift: 3

The area of the apratment is medium and the age of building is old, the apartment tend to be underpriced

- age_old, walking time_close => support: 6%, confidnece: 60%. lift: 3

The age of building is old and it is close to a station, the apartment tend to be underpriced

- room_2, age_old => support: 8%, confidnece: 60%. lift: 3

The number of room is 2 and the age of building is old, the apartment tend to be underpriced

Additionally, from the clustering, cluster 0 and 4, whose residuals are quite low are undervalued. 

So, dense or older area or poor layout apartment tend to be undervalued.


From these answers of small questions, I can answer the major question.

#### Which types of second-hand apartments are ideal for investment.

Based on the integrated analysis using EDA, regression, clustering and freqent pattern mining, the most promising apartments for investment are:

- Older apartment located close to stations, which are undervalued despite their convenience
- Apartments with 2 rooms or medium floor size, which has balance between cost and functionality
- Apartments in lower popularity areas, which may be overlooked by the market
- Simple layout properties (missing a Living or Dining), which tend to be priced lower but still meet demand for those who do not have a lot of money

These characteristics appeared clearly in undervalued elements in my analytical approaches. Therefore, investors should look for these types of properties in central Tokyo.

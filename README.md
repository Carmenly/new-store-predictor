# Enhancing Trader Joe's Expansion Strategy with a Data-Driven Location Recommendation System

## **Problem Statement**

As a data scientist at Trader Joe's, I am tasked with developing a recommendation system to identify optimal locations for new store openings across the United States. As the company looks to expand its footprint, determining optimal locations for new stores is crucial for maximizing market penetration and profitability.

The goal of this project is to develop a recommender system that employs a classification model to predict suitable locations for new stores. The model will analyze various factors, including the distribution of existing stores and economic indicators, to identify prime locations for new stores in states that currently do not have any Trader Joe's presence. While the primary focus will be on identifying opportunities in underserved states, the recommender system will be designed to evaluate any area across the country. 


## **Data Dictionary**

| Feature | Type | Description |
| :--- | :--- | :--- |
| zip_code | Integer | zipcodes for all US States |
| Income Data | Float | 56 columns with income ranges for 4 different groups: Households, Families, Married-Couples and Nonfamily Households |
| Year | Integer | The year the income data was collected (2021) |
| store_presence | Integer | Presence or no presence of a Trader Joe's store in a given zipcode |

## **Executive Summary**

The project's objectives are as follows:

* Gather and analyze data pertaining to Trader Joe's store presence and income distribution across the United States.
* Construct a Classification Model capable of accurately predicting which zip codes have a Trader Joe's store presence.
* Implement the Classification Model within a Recommender System designed to identify optimal locations for potential store openings, thereby enhancing strategic decision-making processes.

## *Data Collection*

* Trader Joe's data was obtained by scraping Trader Joe's website.
* US Household income by zip code was obtained from Kaggle (<https://www.kaggle.com/datasets/claygendron/us-household-income-by-zip-code-2021-2011>)
* Geospatial data for zip codes and states was collected to perform spatial analysis using shapefile 
(<https://catalog.data.gov/dataset/tiger-line-shapefile-2022-nation-u-s-2020-census-5-digit-zip-code-tabulation-area-zcta5>)
(<https://www.census.gov/geographies/mapping-files/time-series/geo/carto-boundary-file.html>)


## *Data Cleaning*

* Verified that all zipcodes in the Trader Joe's DataFrame were found in the income DataFrame 
* While Trader Joe's DataFrame had no null values, the income DataFrame had about 20% of its data missing. Missing values were elimninated after ensuring that the removal did not introduce bias. 
* Created a new column "store_presence" in the income DataFrame to reflect the zipcodes with and without store presence 
* The income DataFrame was refined to include only the most recent information available, which pertained to the year 2021.

## *EDA (Exploratory Data Analysis)*

* There are 25,689 zip codes without store presence, whereas only 556 zip codes have at least one store. This distribution underscores the uneven distribution of Trader Joe's stores across zip code areas.
* Analyzed income distribution among Households, Families, Married-Couples and Nonfamily Households.This revealed disparities across the groups, with married-couple families exhibiting higher median incomes compared to nonfamily households.  
* Assessed how income differences influence the precense of a store using the Mean Difference and Absolute Z-score as metrics. Married-Couple households with incomes of 200k or more, non-family households and families earning 200k or more are the top three variables that strongly influence the presence of a store. Conversely, married-couple families with incomes ranging between 50k to 75k significantly influence the absence of store presence.
* Mapped US zipcodes and Trader Joe's stores utilizing Shapefile data. This visualization allowed to discern disparities in store distribution across states. Notably, when comparing California and Texas, despite Texas' larger geographical size, there's a considerable discrepancy in the number of Trader Joe's stores. California exhibits a significantly denser concentration of Trader Joe's compared to Texas.

### *Spatial Features*
* Ensured that both GeoDataFrames ('zips_df' and 'states_df') use the same Coordinate Reference System (CRS) to ensure consistency in spatial representations and accurate spatial operations.
* Performed a spatial left join between the two GeoDataFrames, leveraging the 'intersects' spatial relationship, which allowed for the integration of attributes from 'states_df' into 'zips_df' based on their spatial overlap.
* The resulting GeoDataFrame was saved as 'zips_with_States' to later use in the recommender system. This will provide a comprehensive view of spatial relationships between ZIP codes and states, facilitating more contextually relevant and personalized recommendations based on geographic factors.

## *Modeling*

* Developed a Baseline classification model using XGBoost, achieving an accuracy score of 0.97 and a f1 score of 0.11. Due to the significant class imbalance between instances with and without store presence, the F1 score was chosen as the primary metric for evaluating model performance. 
* Conducted Grid Search to optimize hyperparameters in the XGBoost model, aiming to improve the f1 score. Surprisingly, despite thorough exploration, the f1 score dropped to 0.1.
* Created a function to evaluate the following models: XGBoost, Random Forest and CatBoost Classifiers using gridsearch across different hyperparameters and employing cross-validation. 
* Acknowledging the significant class imbalance in the target variable, implemented downsampling of negative instances at varying ratios: None, 5, 10, 15, and 20.
* The best-performing model identified was a CatBoost Classifier with a target ratio of 10, achieved by downsampling negative classes at a ratio of 10:1. The model's optimal hyperparameters were 'depth': 4, 'iterations': 100, and 'learning_rate': 0.1, resulting in the highest attained F1 score of 0.38. Considering the class imbalance, an F1 score of 0.38 indicates that the model performs relatively well in correctly classifying instances of both classes, despite the uneven distribution between them. 
* The top-performing model was preserved using the "pickle" library, ensuring its accessibility for integration into the recommender system.

## *Recommender System*

* Utilized the best model to estimate the probability of store presence or no store presence per zip code with the predict_proba method. Applied a threshold of 0.5 and converted the probabilities into predictions: a probability of store presence ≥ 0.5 predicts store presence; otherwise, it predicts no store presence. The probabilities of store presence for each zip code were then added to the DataFrame in a column named "probs".
* Performed a left join combining the incomes DataFrame with the zip_with_states GeoDataFrame based on the common zip_code column. The resulting DataFrame incomes_with_states contains all rows from the incomes DataFrame, with additional geographical information, including state data. 
* Created two functions to carry out the recommender system for identifying optimal locations for stores: 
    * The rank_zipcodes_per_state function filters zip codes by a specified state, joins them with additional data including store presence probabilities, and ranks these zip codes based on their probability scores. The resulting ranked list provides insights into which zip codes are most conducive to store establishment. 
    * The plot_state_prob_choropleth function visualizes these probabilities on a choropleth map, enabling users to identify top-ranking zip codes within the state and observe geographical patterns. By combining predictive analytics with geographic visualization, this recommender system offers a data-driven approach to support strategic decision-making in store expansion efforts.
* The function was applied to analyze states lacking store presence: Wyoming, Alaska, West Virginia, South Dakota, North Dakota, Montana, Hawaii and Mississippi.This facilitated the visualization of the top three optimal locations within each state for potential store openings. By leveraging this analysis, stakeholders can identify promising areas for expansion, maximizing strategic decision-making in store development efforts.


## **Recommendations and Limitations**

* Considering the concentration of Trader Joe's stores in a few states, diversifying expansion efforts to target underserved regions could unlock significant growth opportunities. Exploring markets with lower store density may lead to untapped customer segments and increased market share.
* Conducting thorough market research in potential expansion areas is essential. Analyzing demographic, economic, and cultural factors can provide valuable insights into consumer behavior and preferences, facilitating informed decision-making regarding new store locations.
* The majority of stores are concentrated in a few states, such as California and New York. These states present different characteristics than those states with no stores. To address the bias introduced by the concentration of stores in those areas, it's prudent to consider removing these states from the training data before training the model. By doing so, the model can learn from a more balanced dataset that includes representative samples from states with and without store presence. This approach can help mitigate the risk of the model being biased towards states with existing store presence, thus improving its generalization performance on unseen data.


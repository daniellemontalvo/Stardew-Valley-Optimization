# Crop Profitability Analysis

## Methodology

The first part of this project will focus on the crop profitability analysis. In order to do this analysis, I will use the following process:

- Data Collection
    
    I will be using the dataset shared on reddit that contains all the crop information per season. The dataset is missing the cost of these crops so I will add them manually finding the prices on the wiki or in my own game.
    
- Data Cleaning
    
    Clean and preprocess the to ensure consistency and remove any inconsistencies or outliers. This may involve handling missing values, standardizing formats, and converting categorical variables into a usable format.
    
- Exploratory Data Analysis
    
    Conduct EDA to gain insights into the data. Identify patterns, correlations, and trends that could be useful for optimizing gameplay.
    
- Feature Engineering
    
    Create new features that could enhance your analysis. This could involve combining existing features, creating dummy variables for categorical data, or engineering new metrics based on domain knowledge.
    
- Modeling and Optimization
    
    Use machine learning or optimization techniques to build models that help optimize various aspects of gameplay.
    
- Validation and Testing
    
    Test their performance on unseen data to ensure they generalize well and provide useful insights.
    
- Integration and Visuals

## To-do

- [x]  Download the data and prepare the environment for this part of the project in Jupyter.
- [x]  Familiarize myself with the data
- [x]  Perform analysis and find anomalies in the data
    - [x]  Check dimensions
    - [x]  Data types
    - [x]  Outliers
    - [x]  Distributions
    - [x]  Missing values
    - [x]  Invalid values
    - [x]  Duplicated data
- [ ]  

## Notes / Resources

[https://docs.google.com/spreadsheets/d/1uA2IQ_uMLmvEZa64P3ZVMkoGGM8qMqIWAfRYsnyhGRc/edit#gid=157826043](https://docs.google.com/spreadsheets/d/1uA2IQ_uMLmvEZa64P3ZVMkoGGM8qMqIWAfRYsnyhGRc/edit#gid=157826043)

[https://www.youtube.com/watch?v=6lHX7HL6w4U](https://www.youtube.com/watch?v=6lHX7HL6w4U) - Guide to crops

[https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_excel.html](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_excel.html) - Documentation on pandas

# Results

## Data Cleaning

Before starting out, some analysis was performed in order to clean the data for an easier use. Some issues encountered were the following:

- Values in the dataset contained the currency (G), dashes (-) or “days” hardcoded. I ran some code to remove them and convert those values into floats.
- Trees were skewing the data given that it is hard to track their profit given they produce fruits everyday and their sell prices vary based on the quality of the crop. For this reason I decided to remove them. Sweet Gem Berry is the most profitable item and is skweing the data, but for completion reasons in this analysis and since no prediction is being made, I decided to keep it.

```python
change_type_for = ['Seed Price', 'Preserve', 'Keg', 'Days', 'Profit/d' ,'Yield', 'Profit', 'Harvests', 'Reharvest', 'Profit/Season']
for column_name in change_type_for:
    seasons[column_name] = seasons[column_name].astype(str).str.replace("G", "").str.replace("-", '0').astype(float).fillna(0)

#drop trees and empty rows
seasons = seasons.drop([0, 1, 2, 15, 16, 17, 28, 29, 30, 43, 44])
```

## Visualizations

Using the matplotlib package, I plotted the top crops based solely on profit, both grouped by season and overall. I also added a Scatterplot of the seed price vs profit. From the visualizations we can see how fall and spring have the most profitable crops.  Sweet Gem Berry skews the data significantly, but this tells us that it is a very powerful crop to have in the fall.

![Untitled](Crop%20Profitability%20Analysis%20e509af510d39441fb26b132f3585aee9/Untitled.png)

![Untitled](Crop%20Profitability%20Analysis%20e509af510d39441fb26b132f3585aee9/Untitled%201.png)

![Untitled](Crop%20Profitability%20Analysis%20e509af510d39441fb26b132f3585aee9/Untitled%202.png)

### Recommendations

In order to write recommendations i thought it would be much better to use a different metric than profit. Profit is calculated taking the sell price of a regular quality crop and subtracting the seed price. However, there are three different qualities (Regular, Silver, Gold).

To write recommendations I used the average of the three possible sell prices and subtracted the seed price. Then, I divided that number by the amount of days it takes to harvest, since a quicker harvest means you could potentially plant more of that crop and make more total profit at the end of the season.

Based on that analysis, the results were the following:

```python
seasonal_data = seasons.groupby('Season')
recommendations_dfs = []

for season, data in seasonal_data:
    # Rank crops based on total profit over days to harvest
    ranked_crops = data.sort_values(by='Profit_per_day', ascending=False)
    top_planting_recommendations = ranked_crops.head(5)
    
    # Store planting recommendations for the season in a DataFrame
    recommendations_df = pd.DataFrame({
        'Season': [season] * len(top_planting_recommendations),
        'Crop': top_planting_recommendations['Plant'],
        'Profit_Per_Day': top_planting_recommendations['Profit_per_day']
    })
    # Append the DataFrame to the list of recommendations DataFrames
    recommendations_dfs.append(recommendations_df)

# Concatenate all recommendations DataFrames into a single table
all_recommendations_df = pd.concat(recommendations_dfs, ignore_index=True)

# Display the table of recommendations
print("Recommendations for Planting:")
all_recommendations_df
```

![Untitled](Crop%20Profitability%20Analysis%20e509af510d39441fb26b132f3585aee9/Untitled%203.png)

From this output we can see that the list changed a little when compared to the plots we did merely based on profit.
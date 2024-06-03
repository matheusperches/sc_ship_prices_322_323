
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/matheus-perches/)

#### [Return to my Portfolio](https://github.com/matheusperches/matheusperches.github.io) 

## [Ship Price Comparison - Data Exploration in R]([https://github.com/matheusperches/PlaygroundProj](https://github.com/matheusperches/sc_ship_prices_322_323))

#### Gathering the data 
- I obtained the data from the website [https://scfocus.org/ship-sale-rental-locations-history/]. It is a community maintained and trusted website with various resources and information about the game.

#### Data Cleaning

- Cleaning was done in Google Sheets, using formulas to format the fields.
- 1st step: Remove dots and brackets from the cells. The original file contained pricing for both game versions within the same column, and I wanted to change that.
For that, I created the formula:
```
=SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(B2; "("; ""); ")"; ""); ","; ""); "."; "")
```
- Next, I separated the numbers. I thought this would be fairly straightforward, as I had the space between them as the delimiter. However I faced the challenge of getting the number copied to the next cell even if it did not include a space. Because of how the original data was formatted, ship prices that didn't receive an update did not have a second value. So I had to create a condition to account for that.
```
=IF(ISNUMBER(FIND(" "; D2)); SPLIT(D2; " "); TRANSPOSE({D2; D2})
)
```
- After that step, I used the built in tools from Google Sheets to check for duplicated and missing values.
- Then, I created a new sheet with the columns I wanted, standardizing their names to make it easy to manipulate in R. We can do data cleaning in R as well, but I wanted to practice both tools. 

#### Data exploration
- With the data ready to go, I can export the file in a .CSV format and plug it into Kaggle. Kaggle makes it easy for me to create a good looking notebook and to run my code. I can also take advantage of the cloud features to continue working anywhere I want.
- I proceeded to create a new Kaggle project. Then, I could import the data into the input section.
- performed filtering using R code, to obtain the maximum values of ship prices from each version, as well as the top 5 price increases, that would give me some interesting comparisons to do.
- I was particularly interested in knowing how the highest prices faired amongst locations, so I created a graph to display that.
![Most Expensive buyable ship per location](https://raw.githubusercontent.com/matheusperches/sc_ship_prices_322_323/main/per_location_price_increase.png)
- This graph shows the most expensive ship per location, between 3.22 and 3.23. I made sure to add the values next to the end of each bar, allowing the analyst to know the exact number.
- The next graph shows the top 5 biggest ship price increases, and also shows their prices in cases where they were sold among different dealers.
![Top 5 ship price increase](https://raw.githubusercontent.com/matheusperches/sc_ship_prices_322_323/main/top_5_price_increase.png)
- I was interested in knowing which vehicles received the biggest pricing adjustment, so I developed this graph that makes it easy to know, in a percentage format.

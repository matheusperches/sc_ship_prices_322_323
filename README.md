
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/matheus-perches/)

#### [Return to my Portfolio](https://github.com/matheusperches/matheusperches.github.io) 

## [Ship Price Comparison - Data Exploration in R]([https://github.com/matheusperches/PlaygroundProj](https://github.com/matheusperches/sc_ship_prices_322_323))

## Access the complete code [here](https://github.com/matheusperches/sc_ship_prices_322_323/blob/main/star-citizen-ship-list-data-exploration-3-23.ipynb)

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

```R
#Obtaining the highest ship price from version 3.22 
df_max_price_322 <- price_data %>%
group_by(location) %>%
summarize(price_data = max(price_322, na.rm = TRUE))

#Obtaining the highest ship price from version 3.23

df_max_price_323 <- price_data %>%
group_by(location) %>%
summarize(price_data = max(price_323, na.rm = TRUE))

# Finding the most expensive and cheapest ships across both versions to scale the graph.
max_price <- max(price_data$price_322, price_data$price_323)
min_price <- min(price_data$price_322, price_data$price_323)

per_location_price_increase <- ggplot(mapping = aes(x = location)) +
  geom_bar(data = df_max_price_322, aes(y = price_data, fill = "price_322"), width = 0.5, stat = 'identity') +
  geom_bar(data = df_max_price_323, aes(y = price_data, fill = "price_323"), width = 0.5, stat = 'identity', alpha = 0.5) +
  geom_text(data = df_max_price_323, aes(y = price_data, label = sprintf("%.2fM", price_data / 1000000)), vjust = -0.5, hjust = 1.2, size = 3.5, color = "black") +
  geom_text(data = df_max_price_322, aes(y = price_data, label = sprintf("%.2fM", price_data / 1000000)), vjust = -0.5, hjust = 1.2, size = 3.5, color = "black") +
  labs(title = "Most expensive buyable ship by location \n 3.22 vs 3.23",
       x = "Location",
       y = "Price (in millions)",
      fill = "Versions") +
scale_fill_brewer(type = "qual", labels = c("price_322" = "Version 3.22", "price_323" = "Version 3.23")) +
  theme_light() +
  scale_y_continuous(labels = unit_format(unit = "M")) +
  theme(plot.title = element_text(hjust = 0.5)) +
coord_flip()

# Displaying the graph
print(per_location_price_increase)
```

![Most Expensive buyable ship per location](https://raw.githubusercontent.com/matheusperches/sc_ship_prices_322_323/main/per_location_price_increase.png)
- This graph shows the most expensive ship per location, between 3.22 and 3.23. I made sure to add the values next to the end of each bar, allowing the analyst to know the exact number.
- There was a signficant price increase from the most expensive ship.

```R
# Finding the ship which had the biggest price increase from 3.22 to 3.23 

# Calculate price change
ship_price_delta <- price_data %>%
  arrange(ship) %>%
  mutate(price_increase = price_323 - price_322)

max_price_delta <- ship_price_delta %>%
  slice(which.max(price_increase))  # Retrieve the row with the maximum price increase

# Selecting the top 5 vehicles to avoid graph clutter

top_5 <- ship_price_delta %>%
  arrange(desc(price_increase)) %>%
    head(6)

# Calculating the percentage of the price increase on the top 5 vehicles

top_5_percent <- top_5 %>%
  mutate(percentage_increase = ((price_323 - top_5$price_322) / top_5$price_322 * 100) / 100)  %>%
arrange(desc(percentage_increase))
```

- This sets up the data to be plotted in the graph neatly.

```R
# Finding the max percentage increase
max_percent_delta <- max(top_5_percent$percentage_increase)


# Plotting the graph
top_5_plot <- ggplot(top_5_percent, aes(x = location, y = percentage_increase, fill = percentage_increase)) +
  geom_bar(stat = "identity", width = 0.4) +
geom_text(aes(label = sprintf("%.2f%%", percentage_increase * 100)), vjust = -0.5) + # Add labels at the top of each bar
  labs(title = "Top 5 ship price increases, relative to 3.22",
       x = "Location",
       y = "Price Increase (%)") +
  theme(plot.title = element_text(hjust = 0.5)) +
scale_y_continuous(labels = scales::percent_format(accuracy = 0.1),
                  expand = expansion(mult = c(0, 0.1))) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) + 
facet_wrap(~ship, scales = "free") +  # Facet by dealer to show each instance separately
 guides(fill = "none")  # Remove the legend for fill (price_increase)

# Displaying the graph
print(top_5_plot)
```
- The next graph shows the top 5 biggest ship price increases, and also shows their prices in cases where they were sold among different dealers.
 ![Top 5 ship price increase](https://raw.githubusercontent.com/matheusperches/sc_ship_prices_322_323/main/top_5_price_increase.png)
- I was interested in knowing which vehicles received the biggest pricing adjustment, so I developed this graph that makes it easy to know, in a percentage format.
- An interesting remark about this graph is that 3 out of the top 5 ships were all Hercules series.

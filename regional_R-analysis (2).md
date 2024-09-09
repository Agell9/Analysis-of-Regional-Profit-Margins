```R
#load the data 
Profits <- read_csv("Sales-Profit_portfolio_project.csv")
View(Profits)

# 1. Determine the number of high-margin products (above 38% profit margin)
# needed to be sold per category and per region to increase total profit by 10%.

# Filter products with a profit margin greater than 38%
high_margin_products <- Profits[Profits$profit.margin > 38, ]

# Aggregate current profit by Category and Region
current_profit <- aggregate(high_margin_products$profit.per.unit * high_margin_products$Quantity,
                            by = list(Category = high_margin_products$Category, Region = high_margin_products$Region),
                            FUN = sum)
colnames(current_profit)[3] <- "Total_Profit"

# Calculate the total current profit
total_current_profit <- sum(current_profit$Total_Profit)


# Set the target profit to achieve a 10% increase
target_profit <- total_current_profit * 1.10


# Calculate the additional profit needed to reach the target
additional_profit_needed <- target_profit - total_current_profit

# Distribute the additional profit needed across categories and regions
current_profit$Additional_Profit_Needed <- current_profit$Total_Profit / total_current_profit * additional_profit_needed


# Determine the number of additional products needed to meet the additional profit
current_profit$Additional_Products_Needed <- current_profit$Additional_Profit_Needed / mean(high_margin_products$profit.per.unit)


# Display the results in a table format
print(current_profit)
kable(current_profit, caption = "Current Profit by Region and Category")


# 2. Analyze the number of low-margin products (below or equal to 38% profit margin)
# that need to be reduced or eliminated to optimize overall profitability.
# Filter products with a profit margin less than or equal to 38%
low_margin_products <- Profits[Profits$profit.margin <= 38, ]


# Aggregate the current loss by Category and Region
current_loss <- aggregate(low_margin_products$profit.per.unit * low_margin_products$Quantity,
                          by = list(Category = low_margin_products$Category, Region = low_margin_products$Region),
                          FUN = sum)
colnames(current_loss)[3] <- "Total_Loss"

# Calculate the number of products that should be reduced or eliminated
current_loss$Products_to_Reduce <- current_loss$Total_Loss / mean(low_margin_products$profit.per.unit)

# Display the results in a table format
print(current_loss)
kable(current_loss, caption = "Number of Products Below 38% Profit Margin to be Reduced or Eliminated by Region and Category")


# 3. Evaluate the impact on overall revenue if high-margin product sales (above 38%)
# are increased by 20% in the most underperforming region.

# Identify the region with the lowest total profit
underperforming_region <- aggregate(high_margin_products$profit.per.unit * high_margin_products$Quantity,
                                    by = list(Region = high_margin_products$Region),
                                    FUN = sum)
colnames(underperforming_region)[2] <- "Total_Profit"
underperforming_region <- underperforming_region[which.min(underperforming_region$Total_Profit), "Region"]

# Focus on high-margin products in the underperforming region
impact_high_margin <- high_margin_products[high_margin_products$Region == underperforming_region, ]

# Project the impact of a 20% increase in sales of these high-margin products
impact_high_margin$Impact_on_Profit <- impact_high_margin$profit.per.unit * impact_high_margin$Quantity * 1.20

# Aggregate the projected profit by Category
projected_profit <- aggregate(impact_high_margin$Impact_on_Profit,
                              by = list(Category = impact_high_margin$Category),
                              FUN = sum)
colnames(projected_profit)[2] <- "New_Total_Profit"

# Aggregate the original profit for comparison
original_profit <- aggregate(high_margin_products$profit.per.unit * high_margin_products$Quantity,
                             by = list(Category = high_margin_products$Category, Region = high_margin_products$Region),
                             FUN = sum)
original_profit <- original_profit[original_profit$Region == underperforming_region, ]
original_profit$Region <- NULL
colnames(original_profit)[2] <- "Original_Total_Profit"

# Compare the original and projected profits
final_comparison <- merge(original_profit, projected_profit, by = "Category")
final_comparison$Profit_Increase <- final_comparison$New_Total_Profit - final_comparison$Original_Total_Profit

# Display the results in a table format
print(final_comparison)
kable(final_comparison, caption = "Comparison of Original vs. Projected Profit After 20% Increase in High-Margin Product Sales")


# 4. Visualize the results with plots.

# Bar plot of total profit by region and category
ggplot(current_profit, aes(x = Region, y = Total_Profit, fill = Category)) +
  geom_bar(stat = "identity") +
  theme_minimal() +
  labs(title = "Total Profit by Region and Category", y = "Total Profit", x = "Region")

# Bar plot of projected profit increase by category in the underperforming region
ggplot(final_comparison, aes(x = Category, y = Profit_Increase, fill = Category)) +
  geom_bar(stat = "identity") +
  theme_minimal() +
  labs(title = "Projected Profit Increase by Category in the Underperforming Region", y = "Profit Increase", x = "Category")

# Pie chart of additional products needed per region
ggplot(current_profit, aes(x = "", y = Additional_Products_Needed, fill = Region)) +
  geom_bar(width = 1, stat = "identity") +
  coord_polar("y", start = 0) +
  theme_minimal() +
  labs(title = "Additional Products Needed per Region", y = "", x = "")
-----------------------------------------------------------------------

```

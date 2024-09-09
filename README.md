# Objective
to identify underperforming regions and product categories. The goal is to increase the sales of high-margin products and strategically reduce or eliminate low-margin products, with the aim of improving overall profitability.

# Dataset 
The dataset was acquired from from kaggle, it has 14 columns and 2500 rows that consist of

ID, Order ID, ship Date, Shipping Method, Category, Sub-Category, Country, Region, 

State, Postal Code, Product Name, Customer, Quantity, Shipping cost

### Data Cleaning:
The dataset was first reviewed in excel  to identify any missing or irrelevant data points.
Columns such as Region, Category, Sales, Profit, and Shipping Cost were retained as key variables for the analysis. A new column was created to calculate the Profit Margin for each product, using the formula: = (Profit / Sales) * 100  This allowed us to understand the percentage of profit relative to the total sales amount for each transaction.

Another column was added to calculate the Profit Per Unit, which helps to measure how much profit was earned per unit sold. The formula used: = Profit / Quantity

After preparing and cleaning the dataset in Excel, the cleaned data was exported and loaded into R for detailed analysis.  This involved further breakdowns by region and category to determine high and low-margin products and perform scenario-based projections, such as determining the impact of increasing high-margin product sales on total profit.

### Key Variables:
- Region: The region where the sales occurred.

- Category: The product category (e.g., Furniture, Office Supplies, etc.).

- Product Name: The name of the product sold.

- Quantity: The number of units sold for the respective product.

- Profit Margin: The percentage of profit relative to the sales for each product.

- Profit Per Unit: The amount of profit earned per unit sold. 

I prefer excel/power Bi for data cleaning because this method provides me with a clear view of the tables, cleaning and aggregate functions  can also be performed in SQL using the following queries:

      SELECT Region, Category, ProductName, Quantity, Sales, Profit, ShippingCost
      FROM Sales-Profit_portfolio_project;

### Create new column for profit margin and profit per unit 
      ALTER TABLE Sales-Profit_portfolio_project
      ADD COLUMN ProfitMargin AS (Profit / Sales) * 100;

      ALTER TABLE Sales-Profit_portfolio_project
      ADD COLUMN ProfitPerUnit AS Profit / Quantity;

# Limatations
One major limitation of this dataset is the absence of Cost of Goods Sold (COGS) data. Without COGS, the calculation of profit is based solely on Sales and Shipping Costs, providing an incomplete picture of actual profitability. This limitation is compounded by varying shipping costs across regions, making it difficult to assess true product profitability consistently. Additionally, the analysis assumes that product costs are uniform, which may not reflect the true costs across different regions and categories.

Another limitation is the lack of Return Data. Returned products can significantly impact the overall profit and performance of a product or category. Without information on product returns, it’s impossible to account for any lost revenue or additional costs incurred from returns, such as restocking fees or shipping costs associated with returns. This can lead to an overestimation of profitability in some product categories or regions.

The data also  does not include information on Promotional Offers or Discounts. Promotions and discounts can have a significant impact on both Sales and Profit Margins. For example, a high volume of sales due to promotions may appear positive, but it may not translate to higher profits if the discounts significantly reduce the profit margin. Without this data, the analysis may overlook key factors that affect sales performance and profitability.

### Load necessary libraries
These libraries are used for data manipulation, visualization, and rendering tables in the analysis.


- For data manipulation- [dplyr](https://dplyr.tidyverse.org/)
          
- For data visualization- [ggplot2](https://ggplot2.tidyverse.org/)

- For reading CSV files- [readr](https://readr.tidyverse.org/)

 - For rendering markdown tables- [knitr](https://yihui.org/knitr/)

- For data cleaning and reshaping- [tidyr](https://tidyr.tidyverse.org/)

# Analysis
###  Load the data 

            Profits <- read_csv("Sales-Profit_portfolio_project.csv")
            View(Profits)


### 1. High-Margin Products Analysis:
The first step of the analysis involved determining the number of high-margin products (those with a profit margin above 38%) that need to be sold in each category and region to achieve a 10% increase in total profit.
Filter products with a profit margin greater than 38%

      high_margin_products <- Profits[Profits$profit.margin > 38, ]


### Aggregate current profit by Category and Region

      current_profit <- aggregate(high_margin_products$profit.per.unit * high_margin_products$Quantity,
                          by = list(Category = high_margin_products$Category, Region = high_margin_products$Region),
                          FUN = sum)
       colnames(current_profit)[3] <- "Total_Profit"
 

**Calculate the total current profit**

      total_current_profit <- sum(current_profit$Total_Profit)


###  Set the target profit to achieve a 10% increase

       target_profit <- total_current_profit * 1.10

### Calculate the additional profit needed to reach the target

      additional_profit_needed <- target_profit - total_current_profit

### Distribute the additional profit needed across categories and regions

      current_profit$Additional_Profit_Needed <- current_profit$Total_Profit / total_current_profit * additional_profit_needed


###  Determine the number of additional products needed to meet the additional profit

       current_profit$Additional_Products_Needed <- current_profit$Additional_Profit_Needed / 
       mean(high_margin_products$profit.per.unit)

### Display the results in a table format

       print(current_profit)
       kable(current_profit,` caption = "Current Profit by Region and Category")


### 2. Low-Margin Products Table Script:
This part of the script analyzes low-margin products that need to be reduced or eliminated to optimize overall profitability:
###  Filter products with a profit margin less than or equal to 38%

      low_margin_products <- Profits[Profits$profit.margin <= 38, ]

### Aggregate the current loss by Category and Region

                          current_loss <- aggregate(low_margin_products$profit.per.unit * low_margin_products$Quantity,
                                    by = list(Category = low_margin_products$Category, Region = low_margin_products$Region),
                                    FUN = sum)
                          colnames(current_loss)[3] <- "Total_Loss"

### Determine the number of products that need to be reduced or eliminated
      current_loss$Products_To_Reduce <- current_loss$Total_Loss / mean(low_margin_products$profit.per.unit)

### Display the results in a table format
      print(current_loss)
      kable(current_loss, caption = "Low-Margin Products by Region and Category")

### 3. Evaluate the impact on overall revenue if high-margin product sales (above 38%) are increased by 20% in the most underperforming region.

 ### Identify the region with the lowest total profit
      underperforming_region <- aggregate(high_margin_products$profit.per.unit * high_margin_products$Quantity,
                                    by = list(Region = high_margin_products$Region),
                                    FUN = sum)
      colnames(underperforming_region)[2] <- "Total_Profit"
      underperforming_region <- underperforming_region[which.min(underperforming_region$Total_Profit), "Region"]

### Focus on high-margin products in the underperforming region
      impact_high_margin <- high_margin_products[high_margin_products$Region == underperforming_region, ]

### Project the impact of a 20% increase in sales of these high-margin products
      impact_high_margin$Impact_on_Profit <- impact_high_margin$profit.per.unit * impact_high_margin$Quantity * 1.20

### Aggregate the projected profit by Category
      projected_profit <- aggregate(impact_high_margin$Impact_on_Profit,
                              by = list(Category = impact_high_margin$Category),
                              FUN = sum)
      colnames(projected_profit)[2] <- "New_Total_Profit"

### Aggregate the original profit for comparison
      original_profit <- aggregate(high_margin_products$profit.per.unit * high_margin_products$Quantity,
                             by = list(Category = high_margin_products$Category, Region = high_margin_products$Region),
                             FUN = sum)
      original_profit <- original_profit[original_profit$Region == underperforming_region, ]
      original_profit$Region <- NULL
      colnames(original_profit)[2] <- "Original_Total_Profit"

### Compare the original and projected profits
      final_comparison <- merge(original_profit, projected_profit, by = "Category")
      final_comparison$Profit_Increase <- final_comparison$New_Total_Profit - final_comparison$Original_Total_Profit

### Display the results in a table format
      print(final_comparison)
      kable(final_comparison, caption = "Comparison of Original vs. Projected Profit After 20% Increase in High-Margin Product Sales")


### 4. Visualize the results with plots.

### Bar plot of total profit by region and category
       ggplot(current_profit, aes(x = Region, y = Total_Profit, fill = Category)) +
        geom_bar(stat = "identity") +
        theme_minimal() +
        labs(title = "Total Profit by Region and Category", y = "Total Profit", x = "Region")

### Bar plot of projected profit increase by category in the underperforming region
        ggplot(final_comparison, aes(x = Category, y = Profit_Increase, fill = Category)) +
        geom_bar(stat = "identity") +
       theme_minimal() +
        labs(title = "Projected Profit Increase by Category in the Underperforming Region", y = "Profit Increase", x = "Category")

### Pie chart of additional products needed per region
      ggplot(current_profit, aes(x = "", y = Additional_Products_Needed, fill = Region)) +
      geom_bar(width = 1, stat = "identity") +
      coord_polar("y", start = 0) +
      theme_minimal() +
      labs(title = "Additional Products Needed per Region", y = "", x = "")
 # Key Insights 
 ### Reports
https://1drv.ms/w/s!AuxKYff9R5xjgatfRx2AA7C4CB_t3A?e=vJCDdq

https://1drv.ms/w/s!AuxKYff9R5xjgapjFH6fUXPiUGcQBw?e=YIY1Kt



### High-Margin Products Analysis:

>  The analysis focused on determining how many additional high-margin products (profit margin > 38%) need to be sold across various categories and regions to achieve a 10% increase in total profit.

> The total current profit across all regions and categories was calculated. The regions and categories requiring the highest number of additional high-margin products were the West region and Office Supplies category, which collectively contribute the most to the current profit.

> To meet the 10% profit increase target, the West Region needs to sell 123 additional high-margin units in the Office Supplies category, generating an additional $1,510.34 in profit.

> Similarly, other regions like the East Region in the Technology category show smaller but significant room for profit growth with a requirement of just 15 additional units to achieve the goal.

### Low-Margin Products Analysis:

>Low-margin products (profit margin ≤ 38%) were identified across all regions. The analysis showed that reducing or eliminating these products could significantly improve overall profitability.

> The West Region in the Office Supplies category was responsible for the largest losses, with $6,577.04 in total losses from low-margin products. Eliminating 951 low-margin units would have the greatest positive impact on profits in this region.

>Other regions like the South Region in Technology also contributed losses, though at a smaller scale.

### Projected Profit Increase from High-Margin Product Sales:

>The Central Region was identified as the most underperforming region, generating the lowest total profit from high-margin products.

>By projecting a 20% increase in sales of high-margin products, the Central Region could increase total profits significantly, especially in the Furniture and Office Supplies categories.

> The projected profit increase for the Furniture category in the Central Region is $756.09, which would contribute to a significant recovery in profitability for this region.
>
> # Power Bi
> https://app.powerbi.com/view?r=eyJrIjoiNGFjM2VjM2MtNTcxOC00NTRiLWE2MGYtYjEzYTQyNjhjYWZkIiwidCI6ImE0MzM3ZTM0LTk0MjktNDQxNS05YjljLTJjNTQ3NmQzYWY1ZSIsImMiOjF9

 

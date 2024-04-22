# Introduction
📊 This project leverages a ☕ coffee shop dataset from Kaggle to perform in-depth analysis aimed at driving data-informed decision-making for 📈 business expansion and 💰 cost optimization. The process involved crafting targeted questions, extensive data exploration, careful data manipulation, and the search for actionable insights and patterns. Ultimately, a compelling 🖼️ dashboard was created to visualize key findings, providing a valuable tool to enhance strategic decision-making.

# Background
☕ Being a coffee addict, My days start with coffee, my breaks need coffee, seomtimes evenings could use a little coffee boost.  It's more than a drink; it's practically my lifeblood. But lately, I've been thinking… there's got to be more to my coffee obsession than just loving the taste. Do I always go to the same coffee shops? Do I have a favorite type of coffee? Does my coffee spending secretly go crazy during certain months?

That's where this sales data comes in! I want to dig in and see what I can uncover. I'm treating this project like an exciting coffee adventure! Let's see what the data says! ☕️

### Data Preview
- transaction_id : Unique sequential ID representing an individual transaction
- transaction_date : Date of the transaction (MM/DD/YY)
- transaction_time : Timestamp of the transaction (HH:MM:SS)
- transaction_qty : Quantity of items sold
- store_id : Unique ID of the coffee shop where the transaction took place
- store_location : Location of the coffee shop where the transaction took place
- product_id : Unique ID of the product sold
- unit_price : Retail price of the product sold
- product_category : Description of the product category
- product_type : Description of the product type
- product_detail : Description of the product detail

### Questions raised
1. When are the busiest hours and days of the week? Can staffing be adjusted for optimal efficiency while ensuring great customer service during peak times?
2. What are the top-selling products overall, by category and by store? Are any strong sellers specific to certain locations?
3. Can you identify segments of customers based on spending habits, product preferences or times they visit?
4. What are the least popular products overall or in specific stores? Can they be removed or reworked into limited-time offers?

# Tools I Used
- **Python**: Perform Data Wrangling and Exploratory Data Analysis (EDA). Searching out what the data could do and what it can uncovers.
- **Power BI**: Develop a dashboard to visualize findings and insights.

# The Analysis

### Import Libraries

```python
import pandas as pd
import matplotlib.pyplot as plt
```

### Data Wrangling

First we check if there's any missing value in the data and observe if that missing values have a value in our future analysis or not. In this case, the data is clean and does not have missing values.
```python
rows_with_null = df[df.isnull().any(axis=1)].copy()
rows_with_null
```

Then we check for duplicate values to remove any unnecessary redundancies. In this data, there's no duplciated values.
```python
duplicates = df[df.duplicated()]
duplicates
```

Here we create new columns such as 'Hour', 'Day', 'Month' and 'Sales' in order to have a better analysis of our data.
```python
df['transaction_time'] = df['transaction_time'].astype(str)
df['Hour'] = df['transaction_time'].str[:2]
df['Hour'] = df['Hour'].str.lstrip('0')  
df['Hour'] = df['Hour'].astype(int)
df['Day'] = df['transaction_date'].dt.day_name()
df['Month'] = df['transaction_date'].dt.month_name()
df['Day_no'] = df['transaction_date'].dt.dayofweek + 1
df['Month_no'] = df['transaction_date'].dt.month
df['Sales'] = df['transaction_qty'] * df['unit_price']
```

In the column 'product_detail', some products have 'Sm', 'Rg' and 'Lg' in the end of their name. This represents the size of drink the customer orders which are Small, Regular and Large. We remove this indicators and create a new column that just contain the individual product's name.
```python
def clean_product_name(name):
    """Removes size indicators (if present) and returns the base product name."""
    words = name.split()  
    if len(words) > 1 and words[-1] in ['Sm', 'Rg', 'Lg']:  # Check for size indicator
        return ' '.join(words[:-1])  # Remove size indicator
    else:
        return name  # Return the original name if no size indicator

# Apply the cleaning function to the Series
df['product_detail_clean'] = df['product_detail'].apply(clean_product_name)

# Get unique main product names
unique_products = df['product_detail_clean'].unique() 
print(unique_products)
```

### The Analysis

1. To recognize peak hours of the coffee shop, we plot a bar chart of number of transactions happen by the hour of the day.
```python
q1 = df.copy()
q1 = q1.groupby(['Hour'])['transaction_id'].count()

plt.figure(figsize=(12, 6))

plt.bar(q1.index, q1.values)
plt.xlabel('Hour of the Day')
plt.ylabel('Number of Transactions')
plt.title('Coffee Shop Transactions by Hour of Day')
plt.xticks(q1.index.to_numpy()) 
plt.tight_layout()
plt.show()
```
(Bar Chart)

Findings and Actions
- Schedule more staff during the 7am-11am period to ensure efficient service and a positive customer experience.
- Customers after 8pm are far less than other hours. Adjust opening hours to 6am-8pm, which means close an hour early. this can reduce some bills and staff could go back earlier than usual.
- Implement special offers between 6am-7am (e.g. coffee + pastry combo deal). This could increase new potential customers and possibly set apart than other cafes.

2. In this code, I create a bar chart for all 3 locations of their best sales by product category. 
```python
# Assuming you have 'store_location', 'product_category' and 'Sales' columns
store_locations = df['store_location'].unique()  

# Plotting setup
n_locations = len(store_locations)

# Create charts for each location
for i, location in enumerate(store_locations):
    fig, ax = plt.subplots(figsize=(12, 6)) # Create a plot for each location

    location_data = df[df['store_location'] == location]  
    q2_c = location_data.groupby('product_category')['Sales'].sum()
    q2_c = q2_c.sort_values(ascending=False)

    ax.bar(q2_c.index, q2_c.values)
    ax.set_xlabel('Product Category')
    ax.set_ylabel('Sales')
    ax.set_title(f'Sales by Product Category (Location: {location})')

    # Set x-axis ticks and rotation
    ax.set_xticks(q2_c.index) # Pass product categories as tick positions
    ax.set_xticklabels(q2_c.index, rotation=45, ha='right')  

    plt.tight_layout()
    plt.show()
```

(Bar Chart)


Findings and Actions
- Coffee and Tea are the top 2 main product that generate the majority of Sales for this coffee shop.
- Upgrade machines that is required to create coffee and tea to maintain and improve quality. Owner also need to make sure these 2 products are not out of stock at all times.
- All 3 stores have packaged chocolate to be the bottom category. Its a coffee shop, chocolate drinks are not the main concern.
- Implement offer if transaction is above certain amount, gave free sample of packaged chocolate. This will expose to customer more about the packaged chocolate.

3. To identify any product preferences in different months, I created a line chart of the Sales of Product Category by Month.
```python
import pandas as pd
import matplotlib.pyplot as plt

# ... your data preparation code 

# Group data by Month and Product Category
monthly_sales = df.groupby(['Month', 'product_category'])['Sales'].sum()

# Unstack and transpose for plotting
monthly_sales_pivoted = monthly_sales.unstack().T 

# Create a line chart with one line per month
plt.figure(figsize=(12, 6))

# Optional styling
line_styles = ['-', '--', '-.', ':']

# Plot lines
for i, (month, data) in enumerate(monthly_sales_pivoted.iterrows()):
    plt.plot(data.index, data.values, label=month, linestyle=line_styles[i % len(line_styles)])

plt.xlabel('Product Category')
plt.ylabel('Sales')
plt.title('Sales by Product Category (Over Time)') 
plt.legend(loc='upper center', bbox_to_anchor=(0.5, -0.15), ncol=3) 
plt.tight_layout()
plt.show()
```
(Line Chart)

Findings and Actions
- By observing this line chart, there's no specific month that other products are more preferred than the other.
- This tells that the owner would not need to stock up specific products for different months.
- For deeper analysis, this data needs to be continue collected for July until December. This could expose us to any seasonal activities that affect product's purchase.


This bar chart shows the total quantity sold of individual product. It depicts which drinks customer prefer's the most.
```python
q3_sold_ovr = df.groupby('product_detail_clean')['transaction_qty'].sum()
q3_sold_ovr = q3_sold_ovr.sort_values(ascending=False)

plt.figure(figsize=(12, 6))

plt.bar(q3_sold_ovr.index, q3_sold_ovr.values)
plt.xlabel('Product Detail')  
plt.ylabel('Quantity Sold') 
plt.title('Quantity Sold by Individual Product') 
plt.xticks(rotation=45, ha='right') 
plt.tight_layout()
plt.show()
```

(Bar Chart)

Findings and Actions
- Aware of top selling products and making sure these products are always in stock.
- Promote best selling products and market them even more in social media to expose them to coffee lovers out there.
- Could collect future data on asking customers whether its the taste or the price that makes them purchase those drinks.

### Dashboard
After completing our analysis, I would like to finalize my findings in a dashboard. This makes insights far easier to understand for non-technical users. Since its about coffee, I make it a coffee theme dashboard ☕

(Dashboard)

# Project: Interactive Data Visualisation Dashboard using Tableau; developed in Python
## Summary
- Collected agriculture/population data compiled from FAO and IPCC between 1990 and 2020 in 236 world regions.
- Converted dataset from CSV to pandas dataframe, cleaned data using Python and stored data from distinct
columns in pair with year recorded in 2D NumPy arrays.
- Performed linear regression on each array using SciPy library and predicted post 2020 values. Saw 87% relative
accuracy in population predictions compared to raw, updated data between 2020 and 2023 from data sources.
- Graphed findings using Matplotlib and Plotly libraries, highlighting statistical outliers.
- Used Tableau to develop an interactive dashboard to view the data and plots with filter feature by continent.
## Project Description
### Agriculture/Population Table Structure
| Column                                   | Description |
| ---------------------------------------- | ----------- |
| Area                                     | The total land area of the country in square kilometers (km²). |
| Year                                     | The year in which the data was recorded. |
| Savanna Fires                            | The total area affected by savanna fires in a year, measured in hectares (ha). |
| Forest Fires                             | The total area affected by forest fires in a year, measured in hectares (ha). |
| Crop Residues                            | The total mass of crop residues generated in a year, measured in metric tonnes (MT). |
| Rice Cultivation                         | The total area of land used for rice cultivation in a year, measured in hectares (ha). |
| Drained organic soils (CO2)              | The amount of carbon dioxide (CO2) emissions from drained organic soils in a year, measured in metric tonnes (MT). |
| Pesticides Manufacturing                 | The total amount of pesticides manufactured in a year, measured in metric tonnes (MT). |
| Food Transport                           | The total carbon dioxide (CO2) emissions from food transport in a year, measured in metric tonnes (MT). |
| Forestland                               | The total area of forestland in the country, measured in hectares (ha). |
| Net Forest conversion                    | The net change in forestland area in a year due to afforestation and deforestation, measured in hectares (ha). |
| Food Household Consumption               | The total amount of food consumed by households in a year, measured in metric tonnes (MT). |
| Food Retail                              | The total carbon dioxide (CO2) emissions from food retail in a year, measured in metric tonnes (MT). |
| On-farm Electricity Use                  | The total amount of electricity used on farms in a year, measured in megawatt-hours (MWh). |
| Food Packaging                           | The total amount of packaging used for food products in a year, measured in metric tonnes (MT). |
| Agrifood Systems Waste Disposal          | The total amount of waste disposed of by the agrifood system in a year, measured in metric tonnes (MT). |
| Food Processing                          | The total carbon dioxide (CO2) emissions from food processing in a year, measured in metric tonnes (MT). |
| Fertilizers Manufacturing                | The total amount of fertilizers manufactured in a year, measured in metric tonnes (MT). |
| IPPU                                     | The total carbon dioxide (CO2) emissions from Industrial Processes and Product Use (IPPU) in a year, measured in metric tonnes (MT). |
| Manure applied to Soils                  | The total amount of manure applied to soils in a year, measured in metric tonnes (MT). |
| Manure left on Pasture                   | The total amount of manure left on pastures in a year, measured in metric tonnes (MT). |
| Manure Management                        | The total carbon dioxide (CO2) emissions from manure management in a year, measured in metric tonnes (MT). |
| Fires in organic soils                   | The total area affected by fires in organic soils in a year, measured in hectares (ha). |
| Fires in humid tropical forests          | The total area affected by fires in humid tropical forests in a year, measured in hectares (ha). |
| On-farm energy use                       | The total energy used on farms in a year, measured in gigajoules (GJ). |
| Rural population                         | The total population living in rural areas of the country. |
| Urban population                         | The total population living in urban areas of the country. |
| Total Population - Male                  | The total male population of the country. |
| Total Population - Female                | The total female population of the country. |
| total_emission                           | The total greenhouse gas emissions of the country in a year, measured in metric tonnes of carbon dioxide equivalent (MT CO2e). |
| Average Temperature °C                   | The average temperature of the country in a year, measured in degrees Celsius (°C). |

- Summary with bullet points


### Technologies Used
- Python Libraries:
    - pandas (Data Cleaning + Preparing Data to be plotted)
    - numpy (Key Data Storage + Fast Data Utilisation with ndarrays)
    - matplotlib/numPy (Linear Regression Plots)
- Tableau (Data Visualisation)
## Important Code Snippets
### Importing Packages/Libraries
<pre>
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import linregress
import pandas as pd

import plotly.express as px
import plotly.offline as pyo
import plotly.graph_objects as go
import plotly.io as pio
</pre>
### Converting dataset from CSV to pandas dataframe
<pre>
  # convert from CSV to pandas df
  df = pd.read_csv("C:\LocalDocs\Summer23\Agrofood_co2_emission.csv")
</pre>
### Clean dataframe of null or '0' values (generic "cleaning" function)
<pre>
  def clean_of_certain_value(valstr):
    for col_count in range(2,14): # Check each column for cells containing undesirable value
        df[key_cols[col_count]].replace(valstr, np.nan, inplace=True) # replace any cells containing undesirable value with NaN
        df.dropna(subset=[key_cols[col_count]], inplace=True)  #drop rows that contain NaN values in current column being checked
        
    df.to_csv("C:\LocalDocs\Summer23\Agrofood_co2_emission.csv", index=False) #write back to file with updated df
</pre>
### Adding a 'continent' column with appropriate values for each row
<pre>
    def add_continent(df):
    df_len = len(df)
    continent_list = []
    wrong_list = []

    for i in range(df_len):
        bol = False
        for continent in continent_dict:
            if df.loc[i, 'Area'] in continent_dict[continent]:
                continent_list.append(continent)
                break

    df['Continent'] = continent_list
</pre>
### Reduce the dataframe to contain only certain "key" columns and group this reduced dataframe by area
<pre>
  key_cols = ['Year', 'Area','Forest fires', 'Food Household Consumption', 'Agrifood Systems Waste Disposal', 'Fertilizers Manufacturing', 
              'Rice Cultivation', 'On-farm Electricity Use', 'On-farm energy use', 'Rural population', 'Urban population',
              'Total Population - Male', 'Total Population - Female', 'Average Temperature °C']
  
  df_len = len(df)
  
  area_list = sorted(set(df.loc[i, 'Area'] for i in range(df_len)))
  
  num_key_cols = len(key_cols)
  
  reduced_df = df[key_cols]
  grouped_reduced_df = reduced_df.groupby('Area')
</pre>
### Pair each of the key columns with year recorded for each country/area (preparing for regression)
<pre>
  area_series = pd.Series({
    area: np.vstack([grouped_reduced_df.get_group(area)[key_cols + ['Year']].values.T]) for area in area_list
})
</pre>
### Plot regression lines in matplotlib with outlier highlighting
<pre>
  def draw_linreg(country, col):
    x = area_series[country][0].flatten()
    y = area_series[country][key_cols.index(col)].flatten()
    # Calculate regression line parameters
    slope, intercept, r_value, p_value, std_err = linregress(x.astype(float), y.astype(float))

    # Predicted values for the given x values
    y_pred = slope * x + intercept

    # Visualize the data and regression line
    plt.scatter(x, y, label='Actual Data', color='blue')
    
    highlight_outliers(x,y)
    
    plt.plot(x, y_pred, 'r', label='Regression Line')
    plt.xlabel('Year')
    plt.ylabel(col)
    plt.title(f"Scatter Plot with Regression Line For {country} in {col}")
    plt.legend()
    plt.show()    
    
def highlight_outliers(dataX, dataY):
    # Calculate the Interquartile Range (IQR)
    q1 = np.percentile(dataY, 25)
    q3 = np.percentile(dataY, 75)
    iqr = q3 - q1

    # Define a threshold for identifying outliers
    threshold = 1.5

    # Identify outliers
    lower_bound = q1 - threshold * iqr 
    upper_bound = q3 + threshold * iqr 
    outliers = dataY[(dataY < lower_bound) | (dataY > upper_bound)]
      
    #highlight the outliers a different colour
    xoutliers = np.array([dataX[np.where(dataY == outlier)] for outlier in outliers])
    plt.scatter(xoutliers, outliers, color='purple', label='Statistical Outlier')
</pre>
### Alternative plotting in plotly.py
<pre>
 def plot_with_regression_line(country, col):
    x = area_series[country][0].flatten()
    y = area_series[country][key_cols.index(col)].flatten()
    # Compute the Pearson correlation coefficient
    corr, _ = pearsonr(x, y)
    
    # Create a scatter plot with or without a linear regression line
    if abs(corr) > 0.8:
        fig = px.scatter(x=x, y=y, labels={'x': 'X', 'y': 'Y'}, trendline='ols')
    else:
        fig = px.scatter(x = area_series[country][0].flatten(), y = area_series[country][key_cols.index(col)].flatten())
        fig.add_trace(go.Scatter(x = area_series[country][0].flatten(), y = area_series[country][key_cols.index(col)].flatten(),
                             mode='lines+markers'))
    
    # Customize the plot
    fig.update_traces(marker=dict(size=10, color='DarkRed', line=dict(width=2, color='Black')), selector=dict(mode='markers'))
    fig.update_layout(
        title = f"Scatter Plot Joined By Lines For {country} in {col}",
        title_font=dict(size=24, color='DarkBlue'),
        xaxis=dict(title='Year', gridcolor='lightgray'),
        yaxis=dict(title=f'{col}', gridcolor='lightgray'),
        plot_bgcolor='white',
        margin=dict(l=0, r=0, b=0, t=0)
    )

    # Show the plot
    fig.show()
</pre>
## Exemplar Plots
- Linear Regression Applied Plots:
    -  https://nathann0209.github.io/pr0j3/plotly_chart1693504259.html
    -  https://nathann0209.github.io/pr0j3/plotly_chart1693518623.html
    -  https://nathann0209.github.io/pr0j3/plotly_chart1693518650.html
- Plots Which Weren't Suitable For Linear Regression:
    -  https://nathann0209.github.io/pr0j3/plotly_chart1693340269.html
    -  https://nathann0209.github.io/pr0j3/plotly_chart1693518614.html
    -  https://nathann0209.github.io/pr0j3/plotly_chart1693518643.html

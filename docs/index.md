# Potential Breakfast Restaurant Locations
## Introduction
Calgary is home to a vibrant culinary scene, and breakfast restaurants are no exception. However, while some areas (such as 17th ave) are nearly saturated in gourmet breakfast establishments, others are nearly devoid of anything other than fast food chains. If an up-and-coming Calgarian chef wanted to establish their very own breakfast joint, which communities would be most suitable? In other words, which communities in Calgary have the highest unmet demand for breakfast? 

## Data
[Foursquare](https://developer.foursquare.com/) was used to gather data on the current selection of breakfast restaurants in Calgary. 

Demographic data on Calgarians and their communities was gathered from [Open Calgary](https://data.calgary.ca/) datasets such as the [2016 Census](https://data.calgary.ca/Demographics/Census-By-Community-2016/cje4-zd6c). The 2016 census data is available as a CSV, and contains many potentially helpful fields:

| Field      | Description             |
-------------|-------------------------|
| CLASS | Type of community (Industrial, Major Park Area, Residential, or Residual Sub Area) |
| EMPLYD_CNT | Employed persons include those 15 years of age and older who are employed full or part time. |
| MF_25_34 | Total number of male and female residents aged 25 to 34 (other age-based fields are similarly helpful) |

The [Community Points](https://data.calgary.ca/Base-Maps/Community-Points/j9ps-fyst) dataset was used to determine the centroids of every community.

The [Traffic Volumes for 2016](https://data.calgary.ca/Transportation-Transit/Traffic-Volumes-for-2016/wtec-i6zq) dataset was later used to determine the average traffic flow through communities. This dataset consists of a series of line segments representing sections of road, each with an average daily weekday traffic (ADWT) figure.

## Methodology
### Acquisition and Processing
After retrieving the census information and merging it with the, requests to Foursquare's `explore` endpoint were made for the centeroids of every community with the search term "breakfast". The resulting list of restaurants was cleaned by removing duplicates by unique ID, and filtering out commonly occuring names of venues (i.e. major quick-serve chains and grocery stores), as the focus of this analysis is on independent breakfast restaurants and small breakfast chains.

The coordinates of each venue were then compared to the community centroids to assign an accurate community code to each venue. After this was done, a dataframe of 367 rows was left, each with a community accurately assigned:

| Community Name	| Community Code | Venue Name	| Venue ID | Lat | Long |
|-----------------|----------------|------------|----------|-----|------|
| MEADOWLARK PARK	| MEA | Phil & Sebastian Coffee Roaster	| 4ca4d97f7f84224b96e5d058	| 51.000426 | -114.073290	|

The venues were then grouped and counted by community code, and the resulting series of quantities was added to the census dataframe:

| Community Code | Breakfast Venues |
|-|-|
|MEA | 6 |

### Exploration
A correlation matrix was generated, but no clear correlations were found with breakfast restaurants:

![A correlation matrix showing no strong correlations](https://hexaguin.github.io/Coursera_Capstone/figs/Breakfast_corr_1.png)

| Variable    | Correlation to Breakfast |
|-------------|-----------|
| MF_5_14     | -0.219541 |
| MF_15_19    | -0.219344 |
| MF_45_54    | -0.167594 |
| CLASS_CODE  | -0.162177 |
| MF_55_64    | -0.157547 |
| MF_65_74    | -0.151949 |
| MF_0_4      | -0.139235 |
| latitude    | -0.125138 |
| RES_CNT     | -0.121087 |
| MF_35_44    | -0.092574 |
| EMPLYD_CNT  | -0.087024 |
| MF_20_24    | -0.061474 |
| MF_75       | -0.041933 |
| longitude   |  0.010927 |
| DWELL_CNT   |  0.018546 |
| MF_25_34    |  0.049030 |
| Breakfast   |  1.000000 |

The negative correlations with residents ages 5-19 and 45-54 were explored, but no major insights were gained from this.

<iframe src="https://hexaguin.github.io/Coursera_Capstone/figs/breakfast_by_age.html" height="650" width="100%"></iframe>
*These plots are interactive. Click and drag to zoom, click a trace in the legend to toggle it, and double click to reset zoom.*

The next exploratory path was to map out the locations of the restaurants and look for geographic patterns:

<iframe src="https://hexaguin.github.io/Coursera_Capstone/figs/breakfast.html" height="650" width="100%"></iframe>

The restaurants were clearly clustered near major roads and other high traffic areas. The 2016 traffic data was imported, and a radius neighbor regressor (similar to K nearest neighbors, but using a fixed radius instead of a fixed quantity of neighbors) was used to generate a traffic heatmap for Calgary:

<iframe src="https://hexaguin.github.io/Coursera_Capstone/figs/traffic.html" height="650" width="100%"></iframe>

Major arteries showed up perfectly well on the heat map (Deerfoot Trail, Glenmore Trail), but there's an inherent flaw in using a standard neighbor regression model (or pretty much any common regression model) to generate a map like this: areas such as downtown consist of many small roads, each with a small to moderate amount of traffic. As a result, the mean traffic quantity is low, even if that area actually has quite a large number of vehicles traveling through it in total. The solution was to write a new implementation of RNR that returns a *sum* of the values of the neighbors instead of the mean. Since it's implemented as a loop, it's much far slower than Scikit-Learn's version, but it works fine for the purposes of this project:

```python
def sum_of_radius(long, lat, radius):
    total = 0
    for index, row in traffic_data.iterrows(): 
        if math.sqrt((row['Latitude']-lat)**2 + (row['Longitude']-long)**2) < radius:
            total += row['VOLUME']
    return total
```

This new model far better represents the parts of the city with high levels of activity.

<iframe src="https://hexaguin.github.io/Coursera_Capstone/figs/traffic_sums.html" height="650" width="100%"></iframe>

Each community had its traffic metric recalculated based on this new sum method. This led to the a potentially solid correlation: traffic and breakfast restaurants had a pearson correlation coefficient of 0.492381.

<iframe src="https://hexaguin.github.io/Coursera_Capstone/figs/breakfast_traffic_sum.html" height="650" width="100%"></iframe>

### Modeling

Polynomial regression was used to model the relationship between traffic in a community and the number of resturants. The relationship, 1.893 + -7.583058934622668\*10<sup>-8</sup>X + 4.6704041130661247\*10<sup>-13</sup>X<sup>2</sup>, can then be compared to the data points:

<iframe src="https://hexaguin.github.io/Coursera_Capstone/figs/poly_regress.html" height="650" width="100%"></iframe>

## Results

### Regression Results

The following communities have significantly fewer breakfast restaurants than should be supported by the traffic they receive:

* Chinatown
* Sunnyside
* Crescent Heights
* Mission
* Rosedale

### Other Findings of Note

* Residential demographics are not a major factor when determining how many breakfast restaurants an area can support
* There is a weak negative correlation between households with children or teens and breakfast restaurants

## Discussion

Prospective breakfast restaurateurs should choose a location based on where their clientelle are going during the day, not where they live. It can be inferred from the data that most morning diners choose breakfast locations on the way to or near their daily destinations (such as workplaces), not near their residences. The negative correlation between children and breakfast restaurants may be because families with kids go out for breakfast less frequently, but the more likely scenario is that both kids and breakfast restaurants share a relationship with traffic flow (kids are often raised in quiet subburbs, breakfast restaurants are located in busy, high-traffic areas). 

Out of the 5 notable outliers:

* Chinatown *may* be a viable option if the restaurateur believes they could create a compelling Chineese breakfast experience. 
* Sunnyside is adjacent to Eau Claire and Hilhurst, both of which have reached the modeled capacity for breakfast restaurants. It also has virtually zero commercial space. Sunnyside is a poor choice of location.
* Crescent Heights has a relatively sizable amount of commercial space on the north side, which also faces the Trans-Canada Highway. There are already a number of restaurants in this region, but no breakfast restaurants. Crescent Heights is an excellent choice of community.
* Like Crescent Heights, Mission is already packed with restaurants. It is also located just off one of Calgary's largest dining destinations, 17th Ave. High traffic, plenty of commercial space, and only a couple competitors. Mission is a good choice.
* Rosedale is directly next to Crescent Heights, and also along the Trans-Canada. However, it is almost entirely housing. There is no practical way to establish a restaurant here.

## Conclusion
Crescent Heights, Mission, and Chinatown are all viable locations to establish a breakfast restaurant. 

# Potential Breakfast Restaurant Locations
## Introduction
Calgary is home to a vibrant culinary scene, and breakfast restaurants are no exception. However, while some areas (such as 17th ave) are nearly saturated in gourmet breakfast establishments, others are nearly devoid of anything other than fast food chains. If an up-and-coming Calgarian chef wanted to establish their very own breakfast joint, which communities would be most suitable? In other words, which communities in Calgary have the highest unmet demand for breakfast? 
## Data
[Foursquare](https://developer.foursquare.com/) will be used to gather data on the current selection of breakfast restaurants in Calgary. Foursquare provides data on different types of restaurants in a given area, as well a number of highly useful statistics on any given restaurant via the [details endpoint](https://developer.foursquare.com/docs/api/venues/details):

* Hours of opperation (`hours`), a key factor in determining whether a restaurant serves breakfast
* Popular times (`popular`), useful for determining if people actually go there for breakfast

Demographic data on Calgarians and their communities will be gathered from [Open Calgary](https://data.calgary.ca/) datasets such as the [2018 Census](https://data.calgary.ca/Demographics/Census-by-Community-2018/cc4n-ndvs). The 2018 census data is available as a CSV, and contains many helpful fields:

| Field      | Description             |
-------------|-------------------------|
| CLASS | Type of community (Industrial, Major Park Area, Residential, or Residual Sub Area) |
| EMPLYD_CNT | Employed persons include those 15 years of age and older who are employed full or part time. |
| MF_25_34 | Total number of male and female residents aged 25 to 34 (other age-based fields are similarly helpful) |

The [Citizen Satisfaction Survey 2017](https://data.calgary.ca/dataset/Citizen-Satisfaction-Survey-2017/kgh7-mhue) will also be used to determine factors such as income category (column q39) on a per-ward basis.

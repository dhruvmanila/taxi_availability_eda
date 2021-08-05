# Taxi Availability EDA

An exploratory data analysis on taxi availability in Singapore

Dataset: Extracted from the API endpoint (https://api.data.gov.sg/v1/transport/taxi-availability) The below function was used to collect the data and store in a CSV file `taxi_availability.csv`.

The Notebook is too large to be uploaded on GitHub, it is available on Google Colab: https://colab.research.google.com/drive/1nw-RLUhurcgXEY73Vq4sXpHnyb6qj1uJ?usp=sharing

```python
import csv
import datetime

import requests

# https://data.gov.sg/dataset/taxi-availability
API_URL = "https://api.data.gov.sg/v1/transport/taxi-availability"


def collect_data():
    # The data we receive is on the nearest datetime from the requested datetime. If we
    # start from minute 0, we might get the data from the previous day resulting in the
    # data started from hour 23 to hour 23 the next day. This could be problematic when
    # either sorting/showing the data.
    #
    # The workaround would be to start roughly a minute later for the day we would like
    # to collect the data from.
    start = datetime.datetime(year=2021, month=8, day=1, hour=0, minute=1, second=0)
    end = datetime.datetime(year=2021, month=8, day=1, hour=23, minute=59, second=59)
    delta = datetime.timedelta(minutes=10)

    with open("taxi_availability.csv", "w") as csvfile:
        fieldnames = ["timestamp", "coordinate", "longitude", "latitude"]
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()
        while start < end:
            print(f"Collecting data on {start}...")
            response = requests.get(API_URL, params={"date_time": start.isoformat()})
            data = response.json()
            features = data["features"][0]
            timestamp = features["properties"]["timestamp"]
            for coordinate in features["geometry"]["coordinates"]:
                writer.writerow(
                    {
                        "timestamp": timestamp,
                        "coordinate": coordinate,
                        "longitude": coordinate[0],
                        "latitude": coordinate[1],
                    }
                )
            start += delta
```

## Number of available taxis per hour

<img alt="available taxis per hour histogram" src="./assets/available_taxis_per_hour.png">

- The number of taxis available are less in the morning and increases in the afternoon and then decreases back again during the midnight.
- As the number of people commuting increases, the supply for the taxis increases with it.


## Time series map of available taxis by the hour

<img alt="static time series map of available taxis" src="./assets/time_series_map.png">

The above is a static image of the animation showcasing the available taxis at every hour on a given day.

## Number of available taxis by the region

<img alt="taxis available in regions" src="./assets/taxis_available_in_regions.png">

- The number of taxis available is more in the following areas:
  - Downtown
  - Airport
  - Tourist spots such as Museum, Garden, etc.
- There are various hotspots all around the city where the availability is more suggesting a common spot for taxis.

## Number of available taxis per hour by the region

### Number of available taxis by the region for hour 0

<img alt="taxis available in regions for hour 0" src="./assets/available_taxis_by_region_hour0.png">

### Number of available taxis by the region for hour 1

<img alt="taxis available in regions for hour 1" src="./assets/available_taxis_by_region_hour1.png">

This is a more granular analysis for number of taxis available by region _per hour_. For more images, please refer to the [Google Colab notebook](https://colab.research.google.com/drive/1nw-RLUhurcgXEY73Vq4sXpHnyb6qj1uJ?usp=sharing).

## Routes taken by the available taxis on an average day

<img alt="most routes taken by available taxis" src="./assets/taxi_routes.png">

TODO: conclusion

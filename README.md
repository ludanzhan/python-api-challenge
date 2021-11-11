# Python & API
## Part I
Creating a Python script to visualize the weather of 500+ cities across the world at varying distances from the equator. Selecting countries by randomly select at least 500 unique (non-repeat) cities based on latitude and longitude 

```python
  # Create a set of random lat and lng combinations
  lats = np.random.uniform(lat_range[0], lat_range[1], size=1500)
  lngs = np.random.uniform(lng_range[0], lng_range[1], size=1500)
  lat_lngs = zip(lats, lngs)

  # Identify nearest city for each lat, lng combination
  for lat_lng in lat_lngs:
      city = citipy.nearest_city(lat_lng[0], lat_lng[1]).city_name
    
      # If the city is unique, then add it to a our cities list
      if city not in cities:
          cities.append(city)
```
### Using API to collecting weather data of the randomly selected countris 
```python
  url = "http://api.openweathermap.org/data/2.5/weather?"
  units = "imperial"
  query_url = f"{url}appid={weather_api_key}&units={units}&q="
  
  # Loop Through List of Cities & Perform a Request for Data on Each
  for city in cities:

  # Exception Handling
    try:
        response = requests.get(query_url + city).json()
    except:
        print("City not found. Skipping...") 
    continue

```

### Create Data Frame for collected data

|City 	     |Latitude    |	Longtitude |Max Temp    |	Humidity | Cloudiness  |Wind Speed 	|Country  	|    Date   |
|:---------: |:----------:| :-------:  | :---------:| :-------:| :----------:|:----------:|:---------:|:---------:|
|Tasiilaq 	 |65.6145 	  |-37.6368 	 |32.16 	    |96 	     |100 	       |9.86 	      |GL 	      |1636601351 |
|Lebu 	     |-37.6167 	  |-73.6500 	 |51.62 	    |92 	     |91 	         |7.20 	      |CL 	      |1636601351|
|Mataura 	   |-46.1927 	  |168.8643 	 |75.83 	    |42 	     |100          |4.99 	      |NZ 	      |1636601256|
|Bredasdorp  |-34.5322 	  |20.0403 	   |60.12 	    |88 	     |55           |4.61      	|ZA 	      |1636601352|
|Rikitea 	   |-23.1203 	  |-134.9692 	 |73.89 	    |82 	     |86 	         |14.81 	    |PF 	      |1636601352|

## Plotting Data
### Latitude vs. Temperature Plot
![image](https://github.com/ludanzhan/python-api-challenge/blob/main/WeatherPy/Images/Latitude%20vs%20Max%20Temperature.png)

### Latitude vs. Cloudiness Plot
![image](https://github.com/ludanzhan/python-api-challenge/blob/main/WeatherPy/Images/Latitude%20vs%20Cloudiness%20.png)

### Latitude vs. Wind Speed Plot
![image](https://github.com/ludanzhan/python-api-challenge/blob/main/WeatherPy/Images/Latitude%20vs%20Wind%20Speed%20.png)

### Northern Hemisphere - Max Temp vs. Latitude Linear Regression
![image](https://github.com/ludanzhan/python-api-challenge/blob/main/WeatherPy/Images/north_hemisphere.png)

### Southern Hemisphere - Max Temp vs. Latitude Linear Regression
![image](https://github.com/ludanzhan/python-api-challenge/blob/main/WeatherPy/Images/south_hemisphere.png)

## Part II
Create heat maps use jupyter-gmaps and the Google Places API

### Humidity Heatmap

```python
  # Configure gmaps
  gmaps.configure(api_key=g_key)

  # Use the Lat and Lng as locations and Humidity as the weight
  coordinate = df[["Lat", "Lng"]]

  # store the humidity
  humidity= df["Humidity"]

  fig = gmaps.figure(center=(46.0, -5.0), zoom_level=2)

  heat_layer = gmaps.heatmap_layer(coordinate, weights= humidity, 
                                 dissipating=False, max_intensity=100,
                                 point_radius = 3)

  # Add Heatmap layer to map
  fig.add_layer(heat_layer)
```
![image](https://github.com/ludanzhan/python-api-challenge/blob/main/VacationPy/Images/map.png)

### Hotel Map
* Using Google Places API to search for hotels with 5000 meters of each city

```python
  target_search = "hotel"
  target_radius = 5000
  target_type = "hotel"
  
  # set up a parameters dictionary
  params = {
      "keyword": target_search,
      "radius": target_radius,
      "type": target_type,
      "key": g_key
  }

  base_url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"
```
* iterate through each rows in the data frame to collect data

```python
  for index, row in hotel_df.iterrows():
     # get the latitude and longtitude from df
      lat = row["Lat"]
      lng = row["Lng"]
      city_name = row["City"]
    
      # add keyword to params dict
     params["location"] = f"{lat},{lng}"
    
      # assemble url and make API request
      print(f"Retrieving Results for Index {index}: {city_name}.")
      response = requests.get(base_url, params=params).json()
    
      # extract results
      results = response['results']
```

* Exception Handling
```python
    except(KeyError, IndexError):
        print("Missing field/result... skipping.")
        
    print("------------")
```
* Store the first Hotel result into the DataFrame.
* Plot markers on top of the heatmap.

```python
   hotel_info = [info_box_template.format(**row) for index, row in hotel_df.iterrows()]
  locations = hotel_df[["Lat", "Lng"]]
  marker = gmaps.marker_layer(locations, info_box_content = hotel_info)
  fig.add_layer(marker)
```
![image](https://github.com/ludanzhan/python-api-challenge/blob/main/VacationPy/Images/map2.png)

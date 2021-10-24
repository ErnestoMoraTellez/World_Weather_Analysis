# World_Weather_Analysis

## Purpose

We are helping PLANMYTRIP to create an interfase to help the users find a hotel base on their preferences. We used differnte APIs like open weather and google maps to find the weather of diferent places and to find the locations. We create some maps to identify the places.

### Results

At first we create a random date base with longitudes and latitudes.

  # Create a set of random latitude and longitude combinations.
  lats = np.random.uniform(low=-90.000, high=90.000, size=2000)
  lngs = np.random.uniform(low=-180.000, high=180.000, size=2000)
  lat_lngs = zip(lats, lngs)
  lat_lngs

  # Create a list for holding the cities.
  cities = []
  # Identify the nearest city for each latitude and longitude combination.
  for coordinate in coordinates:
      city = citipy.nearest_city(coordinate[0], coordinate[1]).city_name

      # If the city is unique, then we will add it to the cities list.
      if city not in cities:
          cities.append(city)
  # Print the city count to confirm sufficient count.

Using dataframes we set one with all the cities near to the lat and lng data.

  # Create an empty list to hold the weather data.
  city_data = []
  # Loop through all the cities in the list.
  record_count = 1
  set_count = 1

  for i, city in enumerate(cities):

      # Group cities in sets of 50 for logging purposes.
      if (i % 50 == 0 and i >= 50):
          set_count += 1
          record_count = 1
      # Create endpoint URL with each city.
      city_url = url + "&q=" + city.replace(" ","+")

      # Log the URL, record, and set numbers and the city.
      print(f"Processing Record {record_count} of Set {set_count} | {city}")
      # Add 1 to the record count.
      record_count += 1

      # Run an API request for each of the cities.
      try:
          # Parse the JSON and retrieve data.
          city_weather = requests.get(city_url).json()
          # Parse out the needed data.
          city_lat = city_weather["coord"]["lat"]
          city_lng = city_weather["coord"]["lon"]
          city_max_temp = city_weather["main"]["temp_max"]
          city_humidity = city_weather["main"]["humidity"]
          city_clouds = city_weather["clouds"]["all"]
          city_wind = city_weather["wind"]["speed"]
          city_country = city_weather["sys"]["country"]
          city_current_description = city_weather["weather"][0]["description"]
          # Append the city information into city_data list.
          city_data.append({"City": city.title(),
                            "Lat": city_lat,
                            "Lng": city_lng,
                            "Max Temp": city_max_temp,
                            "Humidity": city_humidity,
                            "Cloudiness": city_clouds,
                            "Wind Speed": city_wind,
                            "Country": city_country,
                            "Current Description": city_current_description})

  # If an error is experienced, skip the city.
      except:
          print("City not found. Skipping...")
          pass

Then we save all the information in a CSV

  # Create the output file (CSV).
  output_data_file = "WeatherPy_Database.csv"
  # Export the City_Data into a CSV.
  city_data_df.to_csv(output_data_file, index_label="City_ID")

After getting all the cities, we ask the user for the temperature they like so we can select the citites that match with the user selecction.

  # 2. Prompt the user to enter minimum and maximum temperature criteria
  min_temp = float(input("What is the minimum temperature you would like for your trip? "))
  max_temp = float(input("What is the maximum temperature you would like for your trip? "))

Then we find somre hotel near the city, to give the user an option.

  # 6a. Set parameters to search for hotels with 5000 meters.
  params = {
      "radius": 5000,
      "type": "lodging",
      "key": g_key
  }

  # 6b. Iterate through the hotel DataFrame.
  for index, row in hotel_df.iterrows():
      # 6c. Get latitude and longitude from DataFrame
      lat = row["Lat"]
      lng = row["Lng"]
      params["location"] = f"{lat},{lng}"
      # 6d. Set up the base URL for the Google Directions API to get JSON data.
      base_url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"

      # 6e. Make request and retrieve the JSON data from the search. 
      hotels = requests.get(base_url, params=params).json()

      # 6f. Get the first hotel from the results and store the name, if a hotel isn't found skip the city.
      try:
          hotel_df.loc[index, "Hotel Name"] = hotels["results"][0]["name"]
      except (IndexError):
          hotel_df.loc[index, "Hotel Name"] = np.nan
          print("Hotel not found... skipping.")

After getting the hotels, we create a map to show the locations.

![WeatherPy_vacation_map](https://user-images.githubusercontent.com/88845919/138616276-a0a3ca46-b581-4d69-b545-58adffd3c585.PNG)

At the end we select 4  places and create a itinerary. Show the locations and the trip to follow.

Itinerary
![WeatherPy_travel_map](https://user-images.githubusercontent.com/88845919/138616429-b68abf57-bbf2-4d23-a901-e19dee427d19.PNG)

Places information
![WeatherPy_travel_map_markers](https://user-images.githubusercontent.com/88845919/138616447-89b506aa-7f0b-4189-8e5a-ced1a6667834.PNG)

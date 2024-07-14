# Processing Geospatial Data

Libraries and Initial Setup

This code makes use of several powerful libraries to accomplish the task of visualizing bike lanes
in Barcelona and highlighting them based on accident data. Here’s a breakdown of the libraries
used and their purposes:
• requests: This library is used to make HTTP requests to fetch data from the internet.
• zipfile and io: These libraries are used to handle compressed files and manage input/output
operations.
• pandas: A highly versatile library for data manipulation and analysis.
• numpy: Provides support for numerical operations and handling arrays.
• osmnx: A library specifically designed for working with OpenStreetMap data, useful for
extracting and visualizing geographic data.
• sklearn.neighbor: KDTree from this library is used for efficient spatial searches.
• urllib.parse and bs4 (BeautifulSoup): These libraries are used to parse URLs and HTML
content, respectively.
• folium: A library to create interactive maps using Leaflet.js.

Main

The main function initiates the process of downloading, analyzing, and visualizing bike lane safety
data for Barcelona. It first defines the URL for the Zenodo repository containing the data and calls
the get_barcelona_data_url(url) function to retrieve the specific URL for Barcelona’s data. If the
data URL is successfully retrieved, it proceeds to call plot_bike_lanes_with_accidents("barcelona",
barcelona_url). This function downloads the accident data, retrieves the city’s road network,
identifies bike lanes, calculates the proximity of each accident to these lanes, and determines the
density of accidents on each lane. Finally, it creates an interactive map using Folium, highlighting
bike lanes with varying colors based on accident density and marking accident locations. If no
data URL is found, it prints a message indicating the lack of available data for Barcelona. This
process provides a comprehensive visual analysis of bike lane safety in Barcelona.

Accessing the Database

For accessing the database of cycling safety data for various cities, the function get_barcelona_data_url(url)
is designed to retrieve the URL specific to Barcelona. This function begins by sending a GET re-
quest to the specified URL using the requests.get(url) method, which fetches the content of the
webpage at the provided URL. It then checks if the response status code is 200, indicating a suc-
cessful request. If the status code is not 200, it prints "Failed to access URL" and returns None.
If the request is successful, the function proceeds to parse the HTML content of the webpage
using BeautifulSoup(response.content, ’html.parser’). This converts the raw HTML content
into a structured format that allows for easy navigation and extraction of specific elements. The
function then searches for all HTML elements with the tag link and attribute rel=’alternate’ us-
ing soup.find_all(’link’, rel=’alternate’). These elements are typically used to provide alternate
versions of the same content, such as different datasets.
Next, the function iterates over the list of alternate_links to find the one corresponding to
Barcelona. For each link, it extracts the href attribute, which contains the URL of the dataset. It
then parses this URL using urlparse(link[’href’]). The path of the parsed URL is split to isolate
the file name, which is further split to separate the city name.
The funciton, urlparse(link[’href’]).path.split(’/’)[-1].split(’.’)[0] extracts the city name from
the URL. It checks if the extracted city name is ’barcelona’. If it finds a match, it prints "Data
available for: barcelona" and returns the URL of the Barcelona dataset.
If the loop completes without finding a link for Barcelona, the function prints "Barcelona data
not found." and returns None. In cases where the initial GET request fails (response status code is
not 200), the function prints "Failed to access URL." and returns None.
This function efficiently navigates through a webpage containing multiple datasets to pinpoint
and return the URL for the Barcelona dataset, making it easy to access and download the required
data for further processing.

Acquiring Data

For acquiring data related to cycling accidents in Barcelona, the function get_accident_data(city_url,
chosen_city) is designed to download, extract, and process the accident data from a given URL.
This function takes two parameters: city_url, which is the URL where the dataset is located, and
chosen_city, which is the name of the city for which the data is being retrieved.
The function begins by sending a GET request to the provided city_url using requests.get(city_url).
This request attempts to download the content from the specified URL. It then checks if the re-
sponse status code is 200, indicating that the request was successful. If the status code is not 200,
the function prints a message "Failed to access city URL: city_url" and returns None, if the request
is successful, the function proceeds to handle the downloaded content, which is assumed to be a
ZIP file. It uses zipfile.ZipFile(io.BytesIO(response.content)) to read the content of the ZIP file
directly from the response without saving it to disk. This is done by wrapping the response content
in a BytesIO object, allowing zipfile.ZipFile to operate on it as if it were a file.
The function then defines a variable csv_filename to represent the expected name of the CSV
file within the ZIP archive. The expected filename is constructed based on the chosen_city, fol-
lowing the pattern "cycling_safety_chosen_city.csv".
Next, the function iterates over the list of files within the ZIP archive using zip_ref.namelist().
For each file, it checks if the filename ends with the expected CSV filename (csv_filename). If it
finds a match, it opens the file using zip_ref.open(filename) and reads it into a pandas DataFrame
with pd.read_csv(csv_file, delimiter=’,’, encoding=’utf-8’, low_memory=False).
After successfully reading the CSV file into a DataFrame, the function identifies the columns
that contain latitude and longitude data. It searches for these columns by checking if the column
names contain the substrings ’latitude’ and ’longitude’ (case-insensitive) using list comprehen-
sions. If both latitude and longitude columns are found, the function renames them to ’Latitude’
and ’Longitude’ respectively and returns a DataFrame containing only these two columns.
If the latitude or longitude columns are not found, the function prints "Latitude and Longitude
columns not found in CSV file." and returns None. Additionally, if an error occurs while reading
the CSV file (such as a parsing error), the function catches the exception, prints an error message,
and returns None.
If the loop completes without finding the expected CSV file, the function prints "CSV file not
found for the chosen city." and returns None.
Overall, this function systematically retrieves and processes the accident data for a specified
city, ensuring that the necessary latitude and longitude information is extracted and returned in a
usable format for further analysis.

Calculating Distances

For calculating distances from a point to a line segment, the function point_to_line_distance(x1,
y1, x2, y2, px, py) is designed to determine the shortest distance between a given point (px, py) and
a line segment defined by two endpoints (x1, y1) and (x2, y2). This function uses mathematical
formulas to achieve accurate distance calculations.
First, the function calculates the magnitude of the line segment. This is done using the Eu-
clidean distance formula:

line_mag = √(x2 − x1)^2 + (y2 − y1)^2

This formula computes the straight-line distance between the points (x1, y1) and (x2, y2). If
the magnitude of the line segment (line_mag) is very small (less than a tiny threshold), the function
returns infinity (float(’inf’)). This is a safeguard against degenerate line segments where the start
and end points are essentially the same.
Next, the function calculates the parameter u using the dot product formula:

u = ((px − x1) ∗ (x2 − x1) + (py − y1) ∗ (y2 − y1))/line_mag^2

The parameter u helps in determining the projection of the point (px, py) onto the line defined
by the endpoints. The value of u indicates the relative position of the projection along the line
segment:
• u is less than 0: the projection falls before the start point (x1, y1).
• u is greater than 1: he projection falls beyond the end point (x2, y2).
• u is between 0 and 1: the projection lies within the line segment.
Based on the value of u, the function calculates the closest point on the line segment:
• u < 0: the closest point is the start point (x1, y1).
• u > 1: the closest point is the end point (x2, y2).
• 0 u 1: the closest point is calculated as:

closest_point = (x1 + u ∗ (x2 − x1), y1 + u ∗ (y2 − y1))

Finally, the function computes the distance between the point (px, py) and the closest point on
the line segment. This is again done using the Euclidean distance formula:

distance =√(px − closest_point[0])^2 + (py − closest_point[1])^2

This distance represents the shortest path from the point (px, py) to the line segment, ensuring
an accurate measurement whether the point projects within the segment or outside its endpoints.
The function then returns this computed distance.

Ploting Map

For plotting a map with bike lanes and accidents, the plot_bike_lanes_with_accidents(city_name,
city_url) function begins by retrieving the entire road network graph for the specified city using
OSMnx with ox.graph_from_place(city_name, network_type=’all’). This graph includes all
types of roads, including bike lanes. The function iterates over all the edges in the graph using for
u, v, k, data in G.edges(keys=True, data=True), where each edge represents a road segment be-
tween two nodes, u and v. It checks if the edge represents a bike lane with if data.get(’highway’)
== ’cycleway’. If it’s a bike lane, the coordinates of the endpoints, (x1, y1) and (x2, y2), are
extracted from the graph nodes. A BikeLaneInfo object is created for each identified bike lane and
stored in a list called bike_lanes.
The function then calls get_accident_data(city_url, city_name) to download and extract the
accident data for the specified city. This helper function reads a CSV file containing accident
information, specifically the latitude and longitude coordinates of each accident. The function
iterates over each accident in the retrieved accident data using for accident_index, accident in
accident_data.iterrows(). For each accident, it calculates its distance to each bike lane using the
helper function point_to_line_distance(x1, y1, x2, y2, px, py).
The function determines the minimum distance for each accident to any bike lane and in-
crements the accident count for the closest bike lane if this distance is less than or equal to 0.2
kilometers (200 meters). It then calculates the mean number of accidents per bike lane, ignoring
lanes with zero accidents:

mean_accidents = (∑ accidents)/(number of lanes with accidents)

It computes the density of accidents for each bike lane as a percentage relative to twice the
mean number of accidents:

density = ( accidents/(mean_accidents ∗ 2)) ∗ 100

The function assigns colors to each bike lane based on its accident density:

           green if density < 33
color =    yellow if density < 66
​            red  if density ≥ 66

The function initializes a map centered on Barcelona using folium.Map(location=[41.3851,
2.1734], zoom_start=13), and adds polylines to the map representing bike lanes, with colors
based on their accident densities using folium.PolyLine. It also adds markers at the locations of
accidents using folium.CircleMarker. Finally, the function saves the interactive map to an HTML
file using m.save(’barcelona_bike_lanes.html’). This process generates an interactive map that
visually represents the density of accidents along bike lanes in a city, aiding in the analysis of
cycling safety.



```python
#Analysis
#1 . Interestingly, while there are many high temperatures at the equator, they are not centered there, it looks more like they center just north at the Tropic of Cancer, which makes sense when you realize it is currently summer in the northern hemisphere.
#2. There does not appear to be any correlation between lartitude and cloud coverage, as it is pretty uniformly spready out.
#3. There seems to be a fairly clean split in the humidity across the equator, with the northern hemisphere experiencing moore areas with lower humidity. This could be related to it being summer in the north, and would benefit from analysis from different time frames to explore more fully.
```


```python
#Dependencies
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import time
import requests
import random
from citipy import citipy
import requests as req
import unidecode
from pprint import pprint
```


```python
api_key = "5e3af127b552742b19035a26ffaa0717"

gkey = "AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ"
```


```python
#Build dataframe for random lat+long
location_data = pd.DataFrame()
location_data['random_lat'] = [np.random.uniform(-90,90) for x in range (1500)]
location_data['random_long'] = [np.random.uniform(-180,180) for x in range (1500)]

#add city closest to lat+long blank columns
location_data['closest city'] = ""
location_data['country'] = ""
#location_data.head()
#find closest city/country
for index, row in location_data.iterrows():
    lat = row['random_lat']
    long = row['random_long']
    location_data.set_value(index, 'closest city', citipy.nearest_city(lat,long).city_name)
    location_data.set_value(index, 'country', citipy.nearest_city(lat,long).country_code)

```

    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:14: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      
    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:15: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      from ipykernel import kernelapp as app
    


```python
#drop dupes
location_data = location_data.drop_duplicates(['closest city', 'country'])
location_data = location_data.dropna()

len(location_data['closest city'].value_counts())
#location_data.head()
```




    600




```python
#check for cities that may have the same name in different countries
clone_cities_check = len(location_data['closest city'])
clone_cities_check
```




    602




```python
#cleanup Data for merge
location_data = location_data[['closest city', 'country']]
location_data = location_data.rename(columns = {'closest city': 'city'})
location_data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>bengkulu</td>
      <td>id</td>
    </tr>
    <tr>
      <th>1</th>
      <td>rikitea</td>
      <td>pf</td>
    </tr>
    <tr>
      <th>2</th>
      <td>albany</td>
      <td>au</td>
    </tr>
    <tr>
      <th>3</th>
      <td>hobart</td>
      <td>au</td>
    </tr>
    <tr>
      <th>4</th>
      <td>clyde river</td>
      <td>ca</td>
    </tr>
  </tbody>
</table>
</div>




```python
#create list of 500 cities

#selected_cities = location_data.sample(n=502)
#selected_cities.head()
#len(selected_cities['city'].value_counts())
```


```python
#pull city info from openweathermap from json downloaded, clean to match formatting

city_data = pd.read_json('city.list.json')
#city_data
for index, row in city_data.iterrows():
    lower_case_city = row['name'].lower()
    no_accent_city = unidecode.unidecode(lower_case_city)
    lower_case_country = row['country'].lower()
    no_accent_country = unidecode.unidecode(lower_case_country)
    city_data.set_value(index, 'name',no_accent_city )
    city_data.set_value(index, 'country',lower_case_country )
city_data = city_data.rename(columns = {'name': 'city'})
```

    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:10: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      # Remove the CWD from sys.path while we load stuff.
    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:11: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      # This is added back by InteractiveShellApp.init_path()
    


```python
#merge random cities with location data pulled from openweathermap
merge_df = location_data.merge(city_data, how = 'left', on = ('city' , 'country'))

merge_df = merge_df.drop_duplicates(['city', 'country'])
```


```python
#merge_df.head(55)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>bengkulu</td>
      <td>id</td>
      <td>{'lon': 102.265541, 'lat': -3.80044}</td>
      <td>1649150.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>rikitea</td>
      <td>pf</td>
      <td>{'lon': -134.969177, 'lat': -23.120279}</td>
      <td>4030556.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>albany</td>
      <td>au</td>
      <td>{'lon': 118.123451, 'lat': -34.7099}</td>
      <td>7839657.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>hobart</td>
      <td>au</td>
      <td>{'lon': 147.329407, 'lat': -42.87936}</td>
      <td>2163355.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>clyde river</td>
      <td>ca</td>
      <td>{'lon': -68.591431, 'lat': 70.469162}</td>
      <td>5924351.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>khani</td>
      <td>ru</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>bluff</td>
      <td>nz</td>
      <td>{'lon': 168.333328, 'lat': -46.599998}</td>
      <td>2206939.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>kattivakkam</td>
      <td>in</td>
      <td>{'lon': 80.316673, 'lat': 13.21667}</td>
      <td>1267433.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>qaanaaq</td>
      <td>gl</td>
      <td>{'lon': -69.363213, 'lat': 77.483971}</td>
      <td>3831208.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>carnarvon</td>
      <td>au</td>
      <td>{'lon': 113.633331, 'lat': -24.866671}</td>
      <td>2074865.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>jamestown</td>
      <td>sh</td>
      <td>{'lon': -5.71675, 'lat': -15.93872}</td>
      <td>3370903.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>ostrovnoy</td>
      <td>ru</td>
      <td>{'lon': 39.513062, 'lat': 68.053062}</td>
      <td>556268.0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>east london</td>
      <td>za</td>
      <td>{'lon': 27.911619, 'lat': -33.015289}</td>
      <td>1006984.0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>sembe</td>
      <td>cg</td>
      <td>{'lon': 14.58056, 'lat': 1.64806}</td>
      <td>2255304.0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>saskylakh</td>
      <td>ru</td>
      <td>{'lon': 114.083328, 'lat': 71.916672}</td>
      <td>2017155.0</td>
    </tr>
    <tr>
      <th>17</th>
      <td>hamilton</td>
      <td>bm</td>
      <td>{'lon': -64.783607, 'lat': 32.293892}</td>
      <td>3573198.0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>mahebourg</td>
      <td>mu</td>
      <td>{'lon': 57.700001, 'lat': -20.40806}</td>
      <td>934322.0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>khilok</td>
      <td>ru</td>
      <td>{'lon': 110.467598, 'lat': 51.367401}</td>
      <td>2022463.0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>puerto ayora</td>
      <td>ec</td>
      <td>{'lon': -90.351761, 'lat': -0.73931}</td>
      <td>3652764.0</td>
    </tr>
    <tr>
      <th>22</th>
      <td>souillac</td>
      <td>mu</td>
      <td>{'lon': 57.51667, 'lat': -20.51667}</td>
      <td>933995.0</td>
    </tr>
    <tr>
      <th>23</th>
      <td>port alfred</td>
      <td>za</td>
      <td>{'lon': 26.891041, 'lat': -33.590569}</td>
      <td>964432.0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>ambilobe</td>
      <td>mg</td>
      <td>{'lon': 49.049999, 'lat': -13.2}</td>
      <td>1082243.0</td>
    </tr>
    <tr>
      <th>25</th>
      <td>chokurdakh</td>
      <td>ru</td>
      <td>{'lon': 147.916672, 'lat': 70.633331}</td>
      <td>2126123.0</td>
    </tr>
    <tr>
      <th>26</th>
      <td>nikolskoye</td>
      <td>ru</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>27</th>
      <td>kapaa</td>
      <td>us</td>
      <td>{'lon': -159.318954, 'lat': 22.075211}</td>
      <td>5848280.0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>penzance</td>
      <td>gb</td>
      <td>{'lon': -5.54554, 'lat': 50.105862}</td>
      <td>7294638.0</td>
    </tr>
    <tr>
      <th>30</th>
      <td>bratsk</td>
      <td>ru</td>
      <td>{'lon': 101.614166, 'lat': 56.1325}</td>
      <td>2051523.0</td>
    </tr>
    <tr>
      <th>31</th>
      <td>ushuaia</td>
      <td>ar</td>
      <td>{'lon': -68.300003, 'lat': -54.799999}</td>
      <td>3833367.0</td>
    </tr>
    <tr>
      <th>32</th>
      <td>tutayev</td>
      <td>ru</td>
      <td>{'lon': 39.5406, 'lat': 57.885288}</td>
      <td>480122.0</td>
    </tr>
    <tr>
      <th>33</th>
      <td>aklavik</td>
      <td>ca</td>
      <td>{'lon': -135.010712, 'lat': 68.219131}</td>
      <td>5882953.0</td>
    </tr>
    <tr>
      <th>34</th>
      <td>butaritari</td>
      <td>ki</td>
      <td>{'lon': 172.797394, 'lat': 3.1616}</td>
      <td>7521588.0</td>
    </tr>
    <tr>
      <th>36</th>
      <td>hithadhoo</td>
      <td>mv</td>
      <td>{'lon': 73.083328, 'lat': -0.6000000000000001}</td>
      <td>1282256.0</td>
    </tr>
    <tr>
      <th>37</th>
      <td>kodiak</td>
      <td>us</td>
      <td>{'lon': -152.407227, 'lat': 57.790001}</td>
      <td>5866583.0</td>
    </tr>
    <tr>
      <th>38</th>
      <td>san quintin</td>
      <td>mx</td>
      <td>{'lon': -115.949997, 'lat': 30.48333}</td>
      <td>3984997.0</td>
    </tr>
    <tr>
      <th>39</th>
      <td>kindu</td>
      <td>cd</td>
      <td>{'lon': 25.950001, 'lat': -2.95}</td>
      <td>212902.0</td>
    </tr>
    <tr>
      <th>40</th>
      <td>busselton</td>
      <td>au</td>
      <td>{'lon': 115.370796, 'lat': -33.684769}</td>
      <td>7839477.0</td>
    </tr>
    <tr>
      <th>42</th>
      <td>arraial do cabo</td>
      <td>br</td>
      <td>{'lon': -42.027779, 'lat': -22.96611}</td>
      <td>3471451.0</td>
    </tr>
    <tr>
      <th>43</th>
      <td>dikson</td>
      <td>ru</td>
      <td>{'lon': 80.546387, 'lat': 73.506943}</td>
      <td>1507390.0</td>
    </tr>
    <tr>
      <th>44</th>
      <td>vaini</td>
      <td>to</td>
      <td>{'lon': -175.199997, 'lat': -21.200001}</td>
      <td>4032243.0</td>
    </tr>
    <tr>
      <th>45</th>
      <td>hami</td>
      <td>cn</td>
      <td>{'lon': 93.449997, 'lat': 42.799999}</td>
      <td>1529484.0</td>
    </tr>
    <tr>
      <th>46</th>
      <td>akdepe</td>
      <td>tm</td>
      <td>{'lon': 59.378769, 'lat': 42.05513}</td>
      <td>601551.0</td>
    </tr>
    <tr>
      <th>47</th>
      <td>hammerfest</td>
      <td>no</td>
      <td>{'lon': 23.682091, 'lat': 70.663361}</td>
      <td>779683.0</td>
    </tr>
    <tr>
      <th>49</th>
      <td>bredasdorp</td>
      <td>za</td>
      <td>{'lon': 20.04031, 'lat': -34.53215}</td>
      <td>1015776.0</td>
    </tr>
    <tr>
      <th>50</th>
      <td>southbridge</td>
      <td>nz</td>
      <td>{'lon': 172.25, 'lat': -43.816669}</td>
      <td>2182525.0</td>
    </tr>
    <tr>
      <th>51</th>
      <td>mataura</td>
      <td>pf</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>52</th>
      <td>punta arenas</td>
      <td>cl</td>
      <td>{'lon': -70.916672, 'lat': -53.150002}</td>
      <td>3874787.0</td>
    </tr>
    <tr>
      <th>53</th>
      <td>ketchikan</td>
      <td>us</td>
      <td>{'lon': -131.646103, 'lat': 55.34222}</td>
      <td>5554428.0</td>
    </tr>
    <tr>
      <th>54</th>
      <td>bambous virieux</td>
      <td>mu</td>
      <td>{'lon': 57.7575, 'lat': -20.342779}</td>
      <td>1106677.0</td>
    </tr>
    <tr>
      <th>55</th>
      <td>mar del plata</td>
      <td>ar</td>
      <td>{'lon': -57.557541, 'lat': -38.002281}</td>
      <td>3430863.0</td>
    </tr>
    <tr>
      <th>56</th>
      <td>pleasanton</td>
      <td>us</td>
      <td>{'lon': -121.87468, 'lat': 37.66243}</td>
      <td>5383777.0</td>
    </tr>
    <tr>
      <th>59</th>
      <td>mehamn</td>
      <td>no</td>
      <td>{'lon': 27.849171, 'lat': 71.035698}</td>
      <td>778707.0</td>
    </tr>
    <tr>
      <th>60</th>
      <td>grand-santi</td>
      <td>gf</td>
      <td>{'lon': -54.383331, 'lat': 4.25}</td>
      <td>3381538.0</td>
    </tr>
    <tr>
      <th>61</th>
      <td>hermanus</td>
      <td>za</td>
      <td>{'lon': 19.234461, 'lat': -34.418701}</td>
      <td>3366880.0</td>
    </tr>
    <tr>
      <th>62</th>
      <td>provideniya</td>
      <td>ru</td>
      <td>{'lon': -173.300003, 'lat': 64.383331}</td>
      <td>4031574.0</td>
    </tr>
    <tr>
      <th>63</th>
      <td>kavaratti</td>
      <td>in</td>
      <td>{'lon': 72.642029, 'lat': 10.56688}</td>
      <td>1267390.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#fill NA cells with empty string for coordinates
merge_df['coord'] = merge_df['coord'].fillna('') 
merge_df['id'] = merge_df['id'].fillna(0) 
merge_df['id'] = merge_df['id'].astype(dtype = 'int64')
merge_df['id'].dtype
```




    dtype('int64')




```python
#count number of ids returned
#merge_df['id'].value_counts()

no_id = merge_df[merge_df['id'] == 0] 
no_id.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7</th>
      <td>khani</td>
      <td>ru</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>26</th>
      <td>nikolskoye</td>
      <td>ru</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>51</th>
      <td>mataura</td>
      <td>pf</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>69</th>
      <td>codrington</td>
      <td>ag</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>73</th>
      <td>belushya guba</td>
      <td>ru</td>
      <td></td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(no_id)
```




    77




```python
#find Lat + Long for cities missing ID
google_url = 'https://maps.googleapis.com/maps/api/geocode/json?address='

counter= 0
for index, row in merge_df.iterrows():
    if row['id'] == 0:
        city = row['city']
        country = row['country']
        print('getting coordinates for city # %s: %s, %s'% (index, city, country))
        target_url = '%s%s,+%s&key=%s' % (google_url, city, country, gkey)
        print(target_url)
        try:
            response = req.get(target_url).json()
            response_path = response['results'][0]['geometry']['location']
            merge_df.set_value(index, 'coord', {'lon':response_path['lng'], 'lat':response_path['lat']})
        except:
            print("missing data for city # %s: %s, %s" % (index, city, country))
        counter = counter + 1
        
        print(counter)
```

    getting coordinates for city # 7: khani, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=khani,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    

    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:15: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      from ipykernel import kernelapp as app
    

    1
    getting coordinates for city # 26: nikolskoye, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=nikolskoye,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    2
    getting coordinates for city # 51: mataura, pf
    https://maps.googleapis.com/maps/api/geocode/json?address=mataura,+pf&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    3
    getting coordinates for city # 69: codrington, ag
    https://maps.googleapis.com/maps/api/geocode/json?address=codrington,+ag&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    4
    getting coordinates for city # 73: belushya guba, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=belushya guba,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    5
    getting coordinates for city # 74: kaitangata, nz
    https://maps.googleapis.com/maps/api/geocode/json?address=kaitangata,+nz&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    6
    getting coordinates for city # 82: taburi, ph
    https://maps.googleapis.com/maps/api/geocode/json?address=taburi,+ph&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    7
    getting coordinates for city # 86: sobolevo, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=sobolevo,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    8
    getting coordinates for city # 87: barawe, so
    https://maps.googleapis.com/maps/api/geocode/json?address=barawe,+so&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    9
    getting coordinates for city # 95: kokkola, fi
    https://maps.googleapis.com/maps/api/geocode/json?address=kokkola,+fi&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    10
    getting coordinates for city # 102: sentyabrskiy, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=sentyabrskiy,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    missing data for city # 102: sentyabrskiy, ru
    11
    getting coordinates for city # 126: airai, pw
    https://maps.googleapis.com/maps/api/geocode/json?address=airai,+pw&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    12
    getting coordinates for city # 132: macaboboni, ph
    https://maps.googleapis.com/maps/api/geocode/json?address=macaboboni,+ph&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    13
    getting coordinates for city # 138: goderich, sl
    https://maps.googleapis.com/maps/api/geocode/json?address=goderich,+sl&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    14
    getting coordinates for city # 142: nizhneyansk, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=nizhneyansk,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    15
    getting coordinates for city # 148: saleaula, ws
    https://maps.googleapis.com/maps/api/geocode/json?address=saleaula,+ws&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    16
    getting coordinates for city # 151: attawapiskat, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=attawapiskat,+ca&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    17
    getting coordinates for city # 157: taolanaro, mg
    https://maps.googleapis.com/maps/api/geocode/json?address=taolanaro,+mg&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    18
    getting coordinates for city # 159: tabiauea, ki
    https://maps.googleapis.com/maps/api/geocode/json?address=tabiauea,+ki&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    19
    getting coordinates for city # 168: rincon, an
    https://maps.googleapis.com/maps/api/geocode/json?address=rincon,+an&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    20
    getting coordinates for city # 174: bolungarvik, is
    https://maps.googleapis.com/maps/api/geocode/json?address=bolungarvik,+is&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    21
    getting coordinates for city # 192: mocambique, mz
    https://maps.googleapis.com/maps/api/geocode/json?address=mocambique,+mz&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    22
    getting coordinates for city # 193: illoqqortoormiut, gl
    https://maps.googleapis.com/maps/api/geocode/json?address=illoqqortoormiut,+gl&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    23
    getting coordinates for city # 194: tawkar, sd
    https://maps.googleapis.com/maps/api/geocode/json?address=tawkar,+sd&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    24
    getting coordinates for city # 195: samusu, ws
    https://maps.googleapis.com/maps/api/geocode/json?address=samusu,+ws&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    25
    getting coordinates for city # 212: santo domingo, ec
    https://maps.googleapis.com/maps/api/geocode/json?address=santo domingo,+ec&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    26
    getting coordinates for city # 213: lephepe, bw
    https://maps.googleapis.com/maps/api/geocode/json?address=lephepe,+bw&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    27
    getting coordinates for city # 224: tumannyy, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=tumannyy,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    28
    getting coordinates for city # 231: abu kamal, sy
    https://maps.googleapis.com/maps/api/geocode/json?address=abu kamal,+sy&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    29
    getting coordinates for city # 232: marcona, pe
    https://maps.googleapis.com/maps/api/geocode/json?address=marcona,+pe&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    30
    getting coordinates for city # 252: gat, ly
    https://maps.googleapis.com/maps/api/geocode/json?address=gat,+ly&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    31
    getting coordinates for city # 261: grand river south east, mu
    https://maps.googleapis.com/maps/api/geocode/json?address=grand river south east,+mu&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    32
    getting coordinates for city # 274: wahran, dz
    https://maps.googleapis.com/maps/api/geocode/json?address=wahran,+dz&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    33
    getting coordinates for city # 285: kamenskoye, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=kamenskoye,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    34
    getting coordinates for city # 313: tsihombe, mg
    https://maps.googleapis.com/maps/api/geocode/json?address=tsihombe,+mg&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    35
    getting coordinates for city # 337: tillabery, ne
    https://maps.googleapis.com/maps/api/geocode/json?address=tillabery,+ne&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    36
    getting coordinates for city # 355: barentsburg, sj
    https://maps.googleapis.com/maps/api/geocode/json?address=barentsburg,+sj&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    37
    getting coordinates for city # 374: bacuit, ph
    https://maps.googleapis.com/maps/api/geocode/json?address=bacuit,+ph&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    38
    getting coordinates for city # 394: vaitupu, wf
    https://maps.googleapis.com/maps/api/geocode/json?address=vaitupu,+wf&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    39
    getting coordinates for city # 396: amderma, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=amderma,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    40
    getting coordinates for city # 434: acapulco, mx
    https://maps.googleapis.com/maps/api/geocode/json?address=acapulco,+mx&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    41
    getting coordinates for city # 437: marzuq, ly
    https://maps.googleapis.com/maps/api/geocode/json?address=marzuq,+ly&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    42
    getting coordinates for city # 438: mys shmidta, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=mys shmidta,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    43
    getting coordinates for city # 452: temaraia, ki
    https://maps.googleapis.com/maps/api/geocode/json?address=temaraia,+ki&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    44
    getting coordinates for city # 453: faya, td
    https://maps.googleapis.com/maps/api/geocode/json?address=faya,+td&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    45
    getting coordinates for city # 468: gomel, by
    https://maps.googleapis.com/maps/api/geocode/json?address=gomel,+by&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    46
    getting coordinates for city # 494: karkaralinsk, kz
    https://maps.googleapis.com/maps/api/geocode/json?address=karkaralinsk,+kz&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    47
    getting coordinates for city # 503: karauzyak, uz
    https://maps.googleapis.com/maps/api/geocode/json?address=karauzyak,+uz&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    48
    getting coordinates for city # 521: ngukurr, au
    https://maps.googleapis.com/maps/api/geocode/json?address=ngukurr,+au&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    49
    getting coordinates for city # 552: safaga, eg
    https://maps.googleapis.com/maps/api/geocode/json?address=safaga,+eg&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    50
    getting coordinates for city # 555: cumaribo, co
    https://maps.googleapis.com/maps/api/geocode/json?address=cumaribo,+co&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    51
    getting coordinates for city # 557: rungata, ki
    https://maps.googleapis.com/maps/api/geocode/json?address=rungata,+ki&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    52
    getting coordinates for city # 561: obluche, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=obluche,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    missing data for city # 561: obluche, ru
    53
    getting coordinates for city # 574: birin, dz
    https://maps.googleapis.com/maps/api/geocode/json?address=birin,+dz&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    54
    getting coordinates for city # 576: bud, no
    https://maps.googleapis.com/maps/api/geocode/json?address=bud,+no&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    55
    getting coordinates for city # 581: tanete, id
    https://maps.googleapis.com/maps/api/geocode/json?address=tanete,+id&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    56
    getting coordinates for city # 607: bosaso, so
    https://maps.googleapis.com/maps/api/geocode/json?address=bosaso,+so&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    57
    getting coordinates for city # 611: barwaha, in
    https://maps.googleapis.com/maps/api/geocode/json?address=barwaha,+in&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    58
    getting coordinates for city # 613: doctor pedro p. pena, py
    https://maps.googleapis.com/maps/api/geocode/json?address=doctor pedro p. pena,+py&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    59
    getting coordinates for city # 614: hervey bay, au
    https://maps.googleapis.com/maps/api/geocode/json?address=hervey bay,+au&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    60
    getting coordinates for city # 619: avera, pf
    https://maps.googleapis.com/maps/api/geocode/json?address=avera,+pf&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    missing data for city # 619: avera, pf
    61
    getting coordinates for city # 660: cheuskiny, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=cheuskiny,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    missing data for city # 660: cheuskiny, ru
    62
    getting coordinates for city # 663: san carlos, pa
    https://maps.googleapis.com/maps/api/geocode/json?address=san carlos,+pa&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    63
    getting coordinates for city # 666: biak, id
    https://maps.googleapis.com/maps/api/geocode/json?address=biak,+id&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    64
    getting coordinates for city # 670: krasnoselkup, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=krasnoselkup,+ru&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    65
    getting coordinates for city # 708: asau, tv
    https://maps.googleapis.com/maps/api/geocode/json?address=asau,+tv&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    66
    getting coordinates for city # 724: san joaquin, bo
    https://maps.googleapis.com/maps/api/geocode/json?address=san joaquin,+bo&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    67
    getting coordinates for city # 728: phu khieo, th
    https://maps.googleapis.com/maps/api/geocode/json?address=phu khieo,+th&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    68
    getting coordinates for city # 737: lasa, cn
    https://maps.googleapis.com/maps/api/geocode/json?address=lasa,+cn&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    69
    getting coordinates for city # 738: olafsvik, is
    https://maps.googleapis.com/maps/api/geocode/json?address=olafsvik,+is&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    70
    getting coordinates for city # 749: erenhot, cn
    https://maps.googleapis.com/maps/api/geocode/json?address=erenhot,+cn&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    71
    getting coordinates for city # 752: amapa, br
    https://maps.googleapis.com/maps/api/geocode/json?address=amapa,+br&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    72
    getting coordinates for city # 755: jalu, ly
    https://maps.googleapis.com/maps/api/geocode/json?address=jalu,+ly&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    73
    getting coordinates for city # 759: louisbourg, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=louisbourg,+ca&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    74
    getting coordinates for city # 767: fort saint john, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=fort saint john,+ca&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    75
    getting coordinates for city # 801: nacala, mz
    https://maps.googleapis.com/maps/api/geocode/json?address=nacala,+mz&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    76
    getting coordinates for city # 802: madras, in
    https://maps.googleapis.com/maps/api/geocode/json?address=madras,+in&key=AIzaSyDSZO95QfWFgAHVvDFlYZb2LNn9OYlQxnQ
    77
    


```python
merge_df.head(50)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>bengkulu</td>
      <td>id</td>
      <td>{'lon': 102.265541, 'lat': -3.80044}</td>
      <td>1649150</td>
    </tr>
    <tr>
      <th>1</th>
      <td>rikitea</td>
      <td>pf</td>
      <td>{'lon': -134.969177, 'lat': -23.120279}</td>
      <td>4030556</td>
    </tr>
    <tr>
      <th>2</th>
      <td>albany</td>
      <td>au</td>
      <td>{'lon': 118.123451, 'lat': -34.7099}</td>
      <td>7839657</td>
    </tr>
    <tr>
      <th>4</th>
      <td>hobart</td>
      <td>au</td>
      <td>{'lon': 147.329407, 'lat': -42.87936}</td>
      <td>2163355</td>
    </tr>
    <tr>
      <th>6</th>
      <td>clyde river</td>
      <td>ca</td>
      <td>{'lon': -68.591431, 'lat': 70.469162}</td>
      <td>5924351</td>
    </tr>
    <tr>
      <th>7</th>
      <td>khani</td>
      <td>ru</td>
      <td>{'lon': 119.965, 'lat': 56.933333}</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>bluff</td>
      <td>nz</td>
      <td>{'lon': 168.333328, 'lat': -46.599998}</td>
      <td>2206939</td>
    </tr>
    <tr>
      <th>9</th>
      <td>kattivakkam</td>
      <td>in</td>
      <td>{'lon': 80.316673, 'lat': 13.21667}</td>
      <td>1267433</td>
    </tr>
    <tr>
      <th>10</th>
      <td>qaanaaq</td>
      <td>gl</td>
      <td>{'lon': -69.363213, 'lat': 77.483971}</td>
      <td>3831208</td>
    </tr>
    <tr>
      <th>11</th>
      <td>carnarvon</td>
      <td>au</td>
      <td>{'lon': 113.633331, 'lat': -24.866671}</td>
      <td>2074865</td>
    </tr>
    <tr>
      <th>12</th>
      <td>jamestown</td>
      <td>sh</td>
      <td>{'lon': -5.71675, 'lat': -15.93872}</td>
      <td>3370903</td>
    </tr>
    <tr>
      <th>13</th>
      <td>ostrovnoy</td>
      <td>ru</td>
      <td>{'lon': 39.513062, 'lat': 68.053062}</td>
      <td>556268</td>
    </tr>
    <tr>
      <th>14</th>
      <td>east london</td>
      <td>za</td>
      <td>{'lon': 27.911619, 'lat': -33.015289}</td>
      <td>1006984</td>
    </tr>
    <tr>
      <th>15</th>
      <td>sembe</td>
      <td>cg</td>
      <td>{'lon': 14.58056, 'lat': 1.64806}</td>
      <td>2255304</td>
    </tr>
    <tr>
      <th>16</th>
      <td>saskylakh</td>
      <td>ru</td>
      <td>{'lon': 114.083328, 'lat': 71.916672}</td>
      <td>2017155</td>
    </tr>
    <tr>
      <th>17</th>
      <td>hamilton</td>
      <td>bm</td>
      <td>{'lon': -64.783607, 'lat': 32.293892}</td>
      <td>3573198</td>
    </tr>
    <tr>
      <th>19</th>
      <td>mahebourg</td>
      <td>mu</td>
      <td>{'lon': 57.700001, 'lat': -20.40806}</td>
      <td>934322</td>
    </tr>
    <tr>
      <th>20</th>
      <td>khilok</td>
      <td>ru</td>
      <td>{'lon': 110.467598, 'lat': 51.367401}</td>
      <td>2022463</td>
    </tr>
    <tr>
      <th>21</th>
      <td>puerto ayora</td>
      <td>ec</td>
      <td>{'lon': -90.351761, 'lat': -0.73931}</td>
      <td>3652764</td>
    </tr>
    <tr>
      <th>22</th>
      <td>souillac</td>
      <td>mu</td>
      <td>{'lon': 57.51667, 'lat': -20.51667}</td>
      <td>933995</td>
    </tr>
    <tr>
      <th>23</th>
      <td>port alfred</td>
      <td>za</td>
      <td>{'lon': 26.891041, 'lat': -33.590569}</td>
      <td>964432</td>
    </tr>
    <tr>
      <th>24</th>
      <td>ambilobe</td>
      <td>mg</td>
      <td>{'lon': 49.049999, 'lat': -13.2}</td>
      <td>1082243</td>
    </tr>
    <tr>
      <th>25</th>
      <td>chokurdakh</td>
      <td>ru</td>
      <td>{'lon': 147.916672, 'lat': 70.633331}</td>
      <td>2126123</td>
    </tr>
    <tr>
      <th>26</th>
      <td>nikolskoye</td>
      <td>ru</td>
      <td>{'lon': 166.0015368, 'lat': 55.1981604}</td>
      <td>0</td>
    </tr>
    <tr>
      <th>27</th>
      <td>kapaa</td>
      <td>us</td>
      <td>{'lon': -159.318954, 'lat': 22.075211}</td>
      <td>5848280</td>
    </tr>
    <tr>
      <th>28</th>
      <td>penzance</td>
      <td>gb</td>
      <td>{'lon': -5.54554, 'lat': 50.105862}</td>
      <td>7294638</td>
    </tr>
    <tr>
      <th>30</th>
      <td>bratsk</td>
      <td>ru</td>
      <td>{'lon': 101.614166, 'lat': 56.1325}</td>
      <td>2051523</td>
    </tr>
    <tr>
      <th>31</th>
      <td>ushuaia</td>
      <td>ar</td>
      <td>{'lon': -68.300003, 'lat': -54.799999}</td>
      <td>3833367</td>
    </tr>
    <tr>
      <th>32</th>
      <td>tutayev</td>
      <td>ru</td>
      <td>{'lon': 39.5406, 'lat': 57.885288}</td>
      <td>480122</td>
    </tr>
    <tr>
      <th>33</th>
      <td>aklavik</td>
      <td>ca</td>
      <td>{'lon': -135.010712, 'lat': 68.219131}</td>
      <td>5882953</td>
    </tr>
    <tr>
      <th>34</th>
      <td>butaritari</td>
      <td>ki</td>
      <td>{'lon': 172.797394, 'lat': 3.1616}</td>
      <td>7521588</td>
    </tr>
    <tr>
      <th>36</th>
      <td>hithadhoo</td>
      <td>mv</td>
      <td>{'lon': 73.083328, 'lat': -0.6000000000000001}</td>
      <td>1282256</td>
    </tr>
    <tr>
      <th>37</th>
      <td>kodiak</td>
      <td>us</td>
      <td>{'lon': -152.407227, 'lat': 57.790001}</td>
      <td>5866583</td>
    </tr>
    <tr>
      <th>38</th>
      <td>san quintin</td>
      <td>mx</td>
      <td>{'lon': -115.949997, 'lat': 30.48333}</td>
      <td>3984997</td>
    </tr>
    <tr>
      <th>39</th>
      <td>kindu</td>
      <td>cd</td>
      <td>{'lon': 25.950001, 'lat': -2.95}</td>
      <td>212902</td>
    </tr>
    <tr>
      <th>40</th>
      <td>busselton</td>
      <td>au</td>
      <td>{'lon': 115.370796, 'lat': -33.684769}</td>
      <td>7839477</td>
    </tr>
    <tr>
      <th>42</th>
      <td>arraial do cabo</td>
      <td>br</td>
      <td>{'lon': -42.027779, 'lat': -22.96611}</td>
      <td>3471451</td>
    </tr>
    <tr>
      <th>43</th>
      <td>dikson</td>
      <td>ru</td>
      <td>{'lon': 80.546387, 'lat': 73.506943}</td>
      <td>1507390</td>
    </tr>
    <tr>
      <th>44</th>
      <td>vaini</td>
      <td>to</td>
      <td>{'lon': -175.199997, 'lat': -21.200001}</td>
      <td>4032243</td>
    </tr>
    <tr>
      <th>45</th>
      <td>hami</td>
      <td>cn</td>
      <td>{'lon': 93.449997, 'lat': 42.799999}</td>
      <td>1529484</td>
    </tr>
    <tr>
      <th>46</th>
      <td>akdepe</td>
      <td>tm</td>
      <td>{'lon': 59.378769, 'lat': 42.05513}</td>
      <td>601551</td>
    </tr>
    <tr>
      <th>47</th>
      <td>hammerfest</td>
      <td>no</td>
      <td>{'lon': 23.682091, 'lat': 70.663361}</td>
      <td>779683</td>
    </tr>
    <tr>
      <th>49</th>
      <td>bredasdorp</td>
      <td>za</td>
      <td>{'lon': 20.04031, 'lat': -34.53215}</td>
      <td>1015776</td>
    </tr>
    <tr>
      <th>50</th>
      <td>southbridge</td>
      <td>nz</td>
      <td>{'lon': 172.25, 'lat': -43.816669}</td>
      <td>2182525</td>
    </tr>
    <tr>
      <th>51</th>
      <td>mataura</td>
      <td>pf</td>
      <td>{'lon': 168.8660155, 'lat': -46.1934288}</td>
      <td>0</td>
    </tr>
    <tr>
      <th>52</th>
      <td>punta arenas</td>
      <td>cl</td>
      <td>{'lon': -70.916672, 'lat': -53.150002}</td>
      <td>3874787</td>
    </tr>
    <tr>
      <th>53</th>
      <td>ketchikan</td>
      <td>us</td>
      <td>{'lon': -131.646103, 'lat': 55.34222}</td>
      <td>5554428</td>
    </tr>
    <tr>
      <th>54</th>
      <td>bambous virieux</td>
      <td>mu</td>
      <td>{'lon': 57.7575, 'lat': -20.342779}</td>
      <td>1106677</td>
    </tr>
    <tr>
      <th>55</th>
      <td>mar del plata</td>
      <td>ar</td>
      <td>{'lon': -57.557541, 'lat': -38.002281}</td>
      <td>3430863</td>
    </tr>
    <tr>
      <th>56</th>
      <td>pleasanton</td>
      <td>us</td>
      <td>{'lon': -121.87468, 'lat': 37.66243}</td>
      <td>5383777</td>
    </tr>
  </tbody>
</table>
</div>




```python
city_selection = merge_df.sample(n=500)
```


```python
#Now, create columns for weather data, create URL, create loop to go through everything, include print to show loop pis working properly, append data to proper column

weather_counter = 0
city_selection['Temperature'] = ""
city_selection['Humidity'] = ""
city_selection['Cloudiness'] = ""
city_selection['Wind Speed'] = ""
city_selection['Latitude'] = ""
city_selection['Longitude'] = ""
```


```python
city_selection.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
      <th>Temperature</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind Speed</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>355</th>
      <td>barentsburg</td>
      <td>sj</td>
      <td>{'lon': 14.2334597, 'lat': 78.0648475}</td>
      <td>0</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>601</th>
      <td>lagoa</td>
      <td>pt</td>
      <td>{'lon': -8.45974, 'lat': 37.105049}</td>
      <td>8010517</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>356</th>
      <td>sotnikovskoye</td>
      <td>ru</td>
      <td>{'lon': 43.777222, 'lat': 45.003891}</td>
      <td>490108</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>234</th>
      <td>faanui</td>
      <td>pf</td>
      <td>{'lon': -151.75, 'lat': -16.48333}</td>
      <td>4034551</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>138</th>
      <td>goderich</td>
      <td>sl</td>
      <td>{'lon': -13.285784, 'lat': 8.427560999999999}</td>
      <td>0</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
for index, row in city_selection.iterrows():
    print('getting data for city # %s: %s, %s' % (index, row['city'], row['country']))
    if((row['id'])==0 and row['coord'] != ""):
        lat = row['coord']['lat']
        lon = row['coord']['lon']
        query_url = 'https://api.openweathermap.org/data/2.5/weather?lat=%s&lon=%s&APPID=%s&units=imperial' % (lat, lon, api_key)
    elif row['id'] != 0:
        location_id = row['id']
        query_url = 'https://api.openweathermap.org/data/2.5/weather?id=%s&APPID=%s&units=imperial' % (location_id, api_key)
    else:
        weather_city = row['city']
        country = row['country']
        query_url = 'https://api.openweathermap.org/data/2.5/weather?q=%s,%s&APPID=%s&units=imperial' % (row['city'], row['country'], api_key)
    print('URL:') 
    print(query_url)
    
    ###
    try:
        city_weather = requests.get(query_url).json()
        #city_weather
        temp = int(city_weather['main']['temp'])
        humidity = city_weather['main']['humidity']
        cloudiness = city_weather['clouds']['all']
        wind_speed= city_weather['wind']['speed']
        Latitude = int(city_weather['coord']['lat'])
        Longitude = int(city_weather['coord']['lon'])

        city_selection.set_value(index, "Temperature", temp)
        city_selection.set_value(index, "Humidity", humidity)
        city_selection.set_value(index, "Cloudiness", cloudiness)
        city_selection.set_value(index, "Wind Speed", wind_speed)
        city_selection.set_value(index, "Latitude", Latitude)
        city_selection.set_value(index, "Longitude", Longitude)

    except:
        print("Error with data, skipping")

```

    getting data for city # 355: barentsburg, sj
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=78.0648475&lon=14.2334597&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 601: lagoa, pt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=8010517&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    

    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:29: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:30: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:31: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:32: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:33: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:34: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    

    getting data for city # 356: sotnikovskoye, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=490108&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 234: faanui, pf
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4034551&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 138: goderich, sl
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=8.427560999999999&lon=-13.285784&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 119: atuona, pf
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4020109&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 586: nanga eboko, cm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2225457&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 283: xinqing, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2033667&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 697: boende, cd
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=218680&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 121: mersing, my
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1732826&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 174: bolungarvik, is
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=66.1517719&lon=-23.261709&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 344: nome, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5870133&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 553: hanamaki, jp
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2112823&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 220: thompson, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6165406&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 739: kokuy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2021908&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 636: ahuimanu, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5856516&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 728: phu khieo, th
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=16.3763389&lon=102.1287289&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 431: martapura, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1636022&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 304: freeport, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4893171&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 630: monrovia, lr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2274895&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 246: esperance, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2071860&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 623: meadow lake, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6071421&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 631: kayseri, tr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=308463&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 565: aljezur, pt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2271968&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 581: tanete, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-5.337832&lon=120.130402&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 271: cabo san lucas, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3985710&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 348: camacha, pt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2270385&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 232: marcona, pe
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-15.3439659&lon=-75.0844757&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 558: havelock, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4470244&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 147: yago, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3979983&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 221: chumikan, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2025256&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 361: san nicolas, ph
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1688905&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 153: grand gaube, mu
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=934479&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 189: kazachinskoye, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2023029&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 120: the valley, ai
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3573374&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 644: bethel, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4182260&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 575: catuday, ph
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1717787&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 666: biak, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-1.0381022&lon=135.9800848&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 193: illoqqortoormiut, gl
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=70.48556909999999&lon=-21.9628757&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 166: castro, cl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3896218&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 36: hithadhoo, mv
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1282256&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 435: eregli, tr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=315498&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 19: mahebourg, mu
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=934322&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 546: beloha, mg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1067565&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 346: babushkin, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=579870&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 259: longyearbyen, sj
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2729907&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 678: lugovoy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=533618&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 77: isangel, vu
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2136825&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 775: georgetown, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4292686&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 740: waipawa, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2206874&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 727: scarborough, tt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3573703&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 109: yeppoon, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2142316&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 734: roma, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2151187&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 55: mar del plata, ar
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3430863&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 223: aswan, eg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=359792&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 746: karasburg, na
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3356343&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 711: chicama, pe
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3698359&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 494: karkaralinsk, kz
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=49.4129509&lon=75.4772753&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 664: hokitika, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2206894&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 682: aqtobe, kz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=610611&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 613: doctor pedro p. pena, py
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-22.45&lon=-62.34999999999999&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 312: deputatskiy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2028164&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 576: bud, no
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=41.5337325&lon=-87.4994615&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 769: brae, gb
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2654970&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 618: uhlove, ua
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=690820&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 72: khatanga, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2022572&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 196: laguna, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3459094&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 753: saran, fr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6434729&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 669: jasdan, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1269179&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 6: clyde river, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5924351&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 478: liboran, ph
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1706599&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 743: bondo, cd
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=218253&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 767: fort saint john, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=56.252423&lon=-120.846409&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 38: san quintin, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3984997&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 552: safaga, eg
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=26.7500171&lon=33.9359756&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 310: igarka, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1505991&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 536: hilton head island, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4581833&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 61: hermanus, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3366880&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 261: grand river south east, mu
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-20.2888094&lon=57.78141199999999&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 500: brigham city, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5771960&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 162: tazovskiy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1489853&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 4: hobart, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2163355&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 163: corinto, ni
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3620269&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 289: aykhal, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2027296&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 496: pontal do parana, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3478424&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 698: oltu, tr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=741160&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 277: kruisfontein, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=986717&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 643: caruray, ph
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1718234&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 448: moissala, td
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2427697&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 84: dakar, sn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2253354&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 614: hervey bay, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-25.2881539&lon=152.7676633&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 42: arraial do cabo, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3471451&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 415: troitskoye, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=481501&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 577: plettenberg bay, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=964712&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 774: vaitape, pf
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4033077&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 579: bathsheba, bb
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3374083&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 134: batagay, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2027044&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 374: bacuit, ph
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=11.2097089&lon=119.4622539&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 171: avarua, ck
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4035715&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 511: peniche, pt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=8010554&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 438: mys shmidta, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=68.884224&lon=-179.4311219&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 294: pacific grove, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5380437&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 102: sentyabrskiy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?q=sentyabrskiy,ru&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    Error with data, skipping
    getting data for city # 495: porosozero, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=506232&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 456: sikasso, ml
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2451185&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 724: san joaquin, bo
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=37.9832553&lon=-121.8677352&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 264: muyezerskiy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=524100&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 254: namatanai, pg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2090021&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 375: san vicente, ph
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1687409&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 755: jalu, ly
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=29.040621&lon=21.4991334&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 186: lahij, ye
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=73560&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 128: tiksi, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2015306&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 30: bratsk, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2051523&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 130: necochea, ar
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3430443&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 179: farafangana, mg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1065158&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 688: kashan, ir
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=128477&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 124: beringovskiy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2126710&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 86: sobolevo, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=54.300693&lon=155.956757&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 736: kutum, sd
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=371745&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 177: college, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5859699&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 231: abu kamal, sy
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=34.4653258&lon=40.9081831&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 284: aksu, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1529660&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 535: gueret, fr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3014383&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 449: graham, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5795906&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 274: wahran, dz
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=35.6970697&lon=-0.6307988&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 409: mayahi, ne
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2441194&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 751: macherla, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1264647&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 205: kavieng, pg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2094342&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 606: cidreira, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3466165&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 207: arnprior, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5887214&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 181: dickinson, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5688789&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 439: luoyang, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1801797&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 594: solnechnyy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2016305&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 281: chiplun, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1274119&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 17: hamilton, bm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3573198&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 621: aksarka, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1512019&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 404: crotone, it
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6541463&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 481: ishinomaki, jp
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2112576&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 287: leona vicario, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3822745&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 222: grindavik, is
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3416888&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 98: san cristobal, ec
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3651949&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 501: berlevag, no
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=780687&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 251: bardiyah, ly
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=80509&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 563: conde, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3385077&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 557: rungata, ki
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-1.3493599&lon=176.445007&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 434: acapulco, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=16.8531086&lon=-99.8236533&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 82: taburi, ph
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=8.710899399999999&lon=117.3851496&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 230: arlit, ne
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2447513&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 87: barawe, so
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=1.1161954&lon=44.0318162&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 263: sola, vu
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2134814&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 92: tahoua, ne
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2439374&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 504: cumberland, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4352681&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 502: sao filipe, cv
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3374210&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 429: athabasca, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5887916&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 485: ribeira grande, pt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=8010689&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 85: sokolo, ml
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2450849&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 749: erenhot, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=43.653169&lon=111.977943&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 573: riyadh, sa
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=108410&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 764: margate, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=978895&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 7: khani, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=56.933333&lon=119.965&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 679: aracuai, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3471846&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 461: qaqortoq, gl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3420846&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 178: tura, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2014833&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 469: korla, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1529376&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 560: morgan city, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4333811&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 295: banda aceh, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1215501&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 253: coihaique, cl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3894426&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 691: nantucket, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4944903&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 756: mozarlandia, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3456642&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 358: inirida, co
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3671450&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 228: awbari, ly
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2219235&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 43: dikson, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1507390&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 45: hami, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1529484&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 640: narsaq, gl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3421719&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 762: portland, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2152667&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 211: pemangkat, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1631637&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 350: manzhouli, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2035836&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 129: ilulissat, gl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3423146&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 477: porto novo, cv
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3374336&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 217: norwich, gb
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2641181&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 15: sembe, cg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2255304&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 333: biltine, td
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=244878&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 75: victoria, sc
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=241131&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 394: vaitupu, wf
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-13.2308863&lon=-176.1917035&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 187: pisco, pe
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3932145&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 725: letlhakane, bw
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=933535&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 752: amapa, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=0.9019925&lon=-52.0029565&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 301: mimongo, ga
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2398430&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 748: bilibino, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2126682&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 425: quatre cocos, mu
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1106643&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 90: umm lajj, sa
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=100926&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 446: alta floresta, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6323793&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 273: los llanos de aridane, es
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2514651&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 354: nyurba, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2018735&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 735: puerto narino, co
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3671387&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 115: flinders, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2166453&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 523: langsa, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1214724&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 489: antsohihy, mg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1068971&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 474: oktyabrskoye, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=515805&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 74: kaitangata, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-46.28334&lon=169.8470967&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 677: misratah, ly
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2214846&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 209: namibe, ao
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3347019&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 252: gat, ly
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=24.509445&lon=118.1089531&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 157: taolanaro, mg
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-25.0225309&lon=46.9853688&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 580: kokopo, pg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2093685&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 229: chapais, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5919850&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 498: chuy, uy
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3443061&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 149: port macquarie, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2152659&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 680: saquarema, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3448011&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 622: istanbul, tr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=745044&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 681: nchelenge, zm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=175499&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 704: gorontalo, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1643837&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 172: manoharpur, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1263649&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 473: taveta, ke
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=179525&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 352: dingle, ie
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2964782&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 313: tsihombe, mg
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-25.3168473&lon=45.48630929999999&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 293: colesberg, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1013076&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 110: cape town, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3369157&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 708: asau, tv
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-7.488197&lon=178.6807179&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 372: obigarm, tj
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1220888&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 534: grand-lahou, ci
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2288105&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 269: guerrero negro, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4021858&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 125: upernavik, gl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3418910&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 402: wuzhou, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1790840&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 150: baykit, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1510689&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 132: macaboboni, ph
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=16.1952956&lon=119.7802409&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 712: amahai, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1651591&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 53: ketchikan, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5554428&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 155: kotma, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1265964&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 716: midland, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5001929&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 51: mataura, pf
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-46.1934288&lon=168.8660155&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 537: iqaluit, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5983720&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 795: severnoye, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=496381&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 676: half moon bay, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5354943&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 639: thinadhoo, mv
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1337610&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 8: bluff, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2206939&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 400: quang ngai, vn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1568770&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 158: port-cartier, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6111696&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 258: lavrentiya, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4031637&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 516: cortez, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4152003&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 100: mount gambier, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839440&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 151: attawapiskat, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=52.9258846&lon=-82.42889219999999&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 437: marzuq, ly
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=25.552342&lon=15.7820674&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 11: carnarvon, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2074865&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 373: formosa do rio preto, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3463124&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 340: anloga, gh
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2304548&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 520: adrar, dz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2508813&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 503: karauzyak, uz
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=43.0174717&lon=60.011507&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 245: bow island, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5906757&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 218: vardo, no
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=777019&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 185: hit, iq
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=95788&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 213: lephepe, bw
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-23.359949&lon=25.85302&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 286: toamasina, mg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1053384&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 554: tigil, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2120612&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 112: pitimbu, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3391889&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 96: katsuura, jp
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2112309&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 428: kirakira, sb
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2178753&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 555: cumaribo, co
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=4.444497&lon=-69.79868499999999&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 410: lazaro cardenas, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4026082&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 139: maniitsoq, gl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3421982&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 656: solwezi, zm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=897045&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 357: yelabuga, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=468082&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 765: pangnirtung, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6096551&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 70: yar-sale, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1486321&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 521: ngukurr, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-14.7327677&lon=134.7323319&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 562: havre-saint-pierre, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5972291&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 39: kindu, cd
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=212902&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 292: murgab, tm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1218612&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 59: mehamn, no
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=778707&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 585: geraldton, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5960603&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 233: panzhihua, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6929460&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 302: kuching, my
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1735634&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 454: rutana, bi
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=433635&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 111: mehriz, ir
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=124274&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 22: souillac, mu
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=933995&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 637: salamiyah, sy
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=164947&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 567: lasem, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1638352&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 240: phra phutthabat, th
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1607512&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 673: ancud, cl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3899695&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 589: yuzhno-kurilsk, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2119447&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 670: krasnoselkup, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=65.69455599999999&lon=82.525613&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 127: mandalgovi, mn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2030065&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 607: bosaso, so
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=11.2755407&lon=49.1878994&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 700: kalevala, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=554318&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 398: ulundi, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=946257&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 339: tilichiki, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2120591&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 407: marfino, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=529368&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 578: stornoway, gb
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2636790&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 488: pontianak, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1630789&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 741: kerema, pg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2094144&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 526: sterling, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4843650&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 224: tumannyy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=68.878911&lon=35.6675264&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 164: cabedelo, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3404558&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 590: north bend, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5074013&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 460: bandarbeyla, so
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=64814&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 611: barwaha, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=22.2532435&lon=76.0407918&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 336: ntchisi, mw
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=924491&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 290: corowa, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2170260&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 131: rawson, ar
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3839307&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 472: tyret pervaya, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2014705&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 46: akdepe, tm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=601551&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 108: balaghat, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1277661&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 539: nuuk, gl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3421319&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 545: darnah, ly
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=87205&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 655: dolores, ar
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3435038&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 475: saldanha, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3361934&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 257: sept-iles, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6144312&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 703: komsomolskiy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=545728&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 26: nikolskoye, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=55.1981604&lon=166.0015368&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 514: tiarei, pf
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4033356&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 773: petropavlovsk-kamchatskiy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2122104&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 705: pokhara, np
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1282898&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 689: bastia, fr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6448138&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 73: belushya guba, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=71.54555599999999&lon=52.32027799999999&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 314: newport, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4302529&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 616: muisne, ec
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3653967&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 582: anadyr, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2127202&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 466: alaca, tr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=752278&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 617: nanortalik, gl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3421765&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 192: mocambique, mz
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-18.665695&lon=35.529562&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 483: honiara, sb
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2108502&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 62: provideniya, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4031574&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 152: mengen, de
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2871964&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 457: moses lake, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5803990&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 49: bredasdorp, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1015776&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 76: diego de almagro, cl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3892454&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 533: salalah, om
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=286621&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 80: tuktoyaktuk, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6170031&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 744: vila franca do campo, pt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=8010690&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 242: geraldton, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2070998&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 40: busselton, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839477&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 97: hilo, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5855927&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 212: santo domingo, ec
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-0.2389045&lon=-79.17742679999999&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 255: dunedin, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2191562&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 244: hambantota, lk
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1244926&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 549: xinzhi, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1788462&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 371: san juan, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4726440&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 493: prince rupert, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6113406&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 194: tawkar, sd
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=18.425989&lon=37.72748199999999&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 634: westport, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2206900&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 570: boffa, gn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2422968&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 279: sao caetano de odivelas, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3388926&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 798: walcz, pl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3082704&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 13: ostrovnoy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=556268&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 81: ponta do sol, cv
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3374346&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 239: keti bandar, pk
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1174451&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 285: kamenskoye, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=62.46661&lon=166.211243&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 661: muzhi, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1498161&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 463: palm springs, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4167583&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 335: mucurapo, tt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3574069&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 1: rikitea, pf
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4030556&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 270: alvaraes, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3665380&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 445: rundu, na
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3353383&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 219: sant julia de loria, ad
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3039163&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 123: weihai, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1791673&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 627: kefalos, gr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=260308&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 395: djambala, cg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2260401&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 12: jamestown, sh
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3370903&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 161: sembakung, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1627877&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 702: sioux lookout, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6148373&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 144: puerto escondido, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3520994&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 692: alice springs, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839681&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 522: crewe, gb
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2652002&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 556: rath, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1258352&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 32: tutayev, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=480122&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 566: zurrieq, mt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2562250&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 136: saint george, bm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3573062&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 659: vao, nc
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2137773&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 33: aklavik, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5882953&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 345: saint-pierre, pm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3424934&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 738: olafsvik, is
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=64.89604849999999&lon=-23.7060262&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 50: southbridge, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2182525&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 635: grants pass, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5729080&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 452: temaraia, ki
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-3.370417&lon=-168.734039&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 465: sao joao da barra, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3448903&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 210: altay, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1529651&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 183: kaniama, cd
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=214389&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 21: puerto ayora, ec
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3652764&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 167: kachiry, kz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1523662&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 146: port elizabeth, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=964420&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 559: qandala, so
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=53157&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 662: george, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1002145&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 524: australind, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2077454&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 768: lorengau, pg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2092164&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 518: whitianga, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2179103&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 713: darhan, mn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2031966&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 569: sechura, pe
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3691954&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 396: amderma, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=69.751221&lon=61.6636961&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 191: vila velha, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3445026&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 683: miguel hidalgo, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3827408&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 291: ardakan, ir
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=143073&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 487: gambela, et
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=337405&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 513: port hardy, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6111862&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 519: georgetown, sh
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2411397&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 540: nalut, ly
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2214433&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 794: selikhino, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2017042&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 311: malinyi, tz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=155016&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 113: saint-philippe, re
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6690301&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 91: sorland, no
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3137469&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 696: nizhniy ingash, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1497497&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 69: codrington, ag
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=17.6425736&lon=-61.8204456&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 403: hofn, is
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2630299&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 411: fairbanks, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5861897&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 476: izumo, jp
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1861084&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 462: storsteinnes, no
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3136023&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 25: chokurdakh, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2126123&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 571: naze, jp
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1855540&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 241: makarska, hr
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3195890&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 159: tabiauea, ki
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=0.9182079999999999&lon=173.027965&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 588: tautira, pf
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4033557&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 52: punta arenas, cl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3874787&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 551: zhanaozen, kz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=607610&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 770: guarda, pt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=8010533&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 88: mount isa, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2065594&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 750: omboue, ga
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2396853&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 265: mayo, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6068416&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 468: gomel, by
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=52.4411761&lon=30.9878461&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 628: santa rosalia, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3983805&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 60: grand-santi, gf
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3381538&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 399: cockburn town, tc
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3576994&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 414: nishihara, jp
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1855342&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 236: akonolinga, cm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2235776&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 612: syamzha, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=485351&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 668: prado, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3452483&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 9: kattivakkam, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1267433&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 548: abu zabad, sd
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=380348&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 625: kabalo, cd
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=215668&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 397: leh, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1264976&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 208: torbay, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6167817&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 122: banchory, gb
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2656405&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 470: nivala, fi
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=644509&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 95: kokkola, fi
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=63.8415002&lon=23.1250337&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 747: caravelas, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3466980&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 288: vangaindrano, mg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1054329&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 458: bonavista, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5905393&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 608: colares, pt
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=8012548&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 300: la rioja, ar
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3848950&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 170: padang, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1633419&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 694: muros, es
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6357322&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 467: rajula, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1258795&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 547: dunhua, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2037534&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 800: marsh harbour, bs
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3571913&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 406: dwarka, in
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1272140&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 225: tomatlan, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3981391&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 227: te anau, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2181625&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 541: cap malheureux, mu
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=934649&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 168: rincon, an
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=18.3401514&lon=-67.24994590000001&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 297: miri, my
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1738050&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 757: walvis bay, na
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3359638&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 180: menongue, ao
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3347353&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 642: zaraza, ve
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3625066&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 638: koslan, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=544084&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 266: belyy yar, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1510370&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 341: mezhdurechensk, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=527299&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 23: port alfred, za
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=964432&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 760: tuatapere, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2180815&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 31: ushuaia, ar
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3833367&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 480: imbituba, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3461370&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 78: broken hill, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2173911&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 587: kahului, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5847411&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 401: luderitz, na
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3355672&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 709: mbala, zm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=176146&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 34: butaritari, ki
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=7521588&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 515: rabaul, pg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2087894&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 538: xiongzhou, cn
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1788402&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 16: saskylakh, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2017155&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 497: wau, pg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2083653&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 583: wollongong, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839791&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 249: dicabisagan, ph
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1715015&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 99: rivarolo canavese, it
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3169253&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 479: alyangula, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2079582&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 641: salina cruz, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3520064&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 459: kidal, ml
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2455290&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 351: lebu, cl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3883457&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 729: oranjemund, na
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3354071&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 412: buala, sb
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2109528&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 160: hasaki, jp
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2112802&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 801: nacala, mz
    URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-14.5656065&lon=40.6854309&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 260: huejuquilla el alto, mx
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4004616&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 733: talnakh, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1490256&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 510: inyonga, tz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=159134&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 484: norman wells, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6089245&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 237: batagay-alyta, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2027042&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 133: amambai, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3472391&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 272: mazyr, by
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=625324&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 593: cherskiy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2126199&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 107: yellowknife, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6185377&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 247: roblin, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6125860&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 706: maamba, zm
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=908718&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 27: kapaa, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=5848280&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 94: new norfolk, au
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2155415&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 499: terra santa, br
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3386467&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 742: kieta, pg
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2094027&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 331: leningradskiy, ru
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2123814&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 525: kabanjahe, id
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=1214965&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 10: qaanaaq, gl
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3831208&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 198: new london, us
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=4839416&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 360: ahipara, nz
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=2194098&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 154: puerto madryn, ar
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=3840092&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    getting data for city # 303: shellbrook, ca
    URL:
    https://api.openweathermap.org/data/2.5/weather?id=6145951&APPID=5e3af127b552742b19035a26ffaa0717&units=imperial
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
      <th>Temperature</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind Speed</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>355</th>
      <td>barentsburg</td>
      <td>sj</td>
      <td>{'lon': 14.2334597, 'lat': 78.0648475}</td>
      <td>0</td>
      <td>32</td>
      <td>96</td>
      <td>75</td>
      <td>18.34</td>
      <td>78</td>
      <td>14</td>
    </tr>
    <tr>
      <th>601</th>
      <td>lagoa</td>
      <td>pt</td>
      <td>{'lon': -8.45974, 'lat': 37.105049}</td>
      <td>8010517</td>
      <td>62</td>
      <td>93</td>
      <td>0</td>
      <td>4.7</td>
      <td>37</td>
      <td>-8</td>
    </tr>
    <tr>
      <th>356</th>
      <td>sotnikovskoye</td>
      <td>ru</td>
      <td>{'lon': 43.777222, 'lat': 45.003891}</td>
      <td>490108</td>
      <td>57</td>
      <td>29</td>
      <td>0</td>
      <td>13.67</td>
      <td>45</td>
      <td>43</td>
    </tr>
    <tr>
      <th>234</th>
      <td>faanui</td>
      <td>pf</td>
      <td>{'lon': -151.75, 'lat': -16.48333}</td>
      <td>4034551</td>
      <td>81</td>
      <td>100</td>
      <td>56</td>
      <td>14.67</td>
      <td>-16</td>
      <td>-151</td>
    </tr>
    <tr>
      <th>138</th>
      <td>goderich</td>
      <td>sl</td>
      <td>{'lon': -13.285784, 'lat': 8.427560999999999}</td>
      <td>0</td>
      <td>79</td>
      <td>100</td>
      <td>68</td>
      <td>10.54</td>
      <td>8</td>
      <td>-13</td>
    </tr>
    <tr>
      <th>119</th>
      <td>atuona</td>
      <td>pf</td>
      <td>{'lon': -139.033325, 'lat': -9.8}</td>
      <td>4020109</td>
      <td>80</td>
      <td>100</td>
      <td>68</td>
      <td>12.66</td>
      <td>-9</td>
      <td>-139</td>
    </tr>
    <tr>
      <th>586</th>
      <td>nanga eboko</td>
      <td>cm</td>
      <td>{'lon': 12.36667, 'lat': 4.68333}</td>
      <td>2225457</td>
      <td>71</td>
      <td>89</td>
      <td>24</td>
      <td>5.17</td>
      <td>4</td>
      <td>12</td>
    </tr>
    <tr>
      <th>283</th>
      <td>xinqing</td>
      <td>cn</td>
      <td>{'lon': 129.5, 'lat': 48.23333}</td>
      <td>2033667</td>
      <td>56</td>
      <td>73</td>
      <td>0</td>
      <td>5.95</td>
      <td>48</td>
      <td>129</td>
    </tr>
    <tr>
      <th>697</th>
      <td>boende</td>
      <td>cd</td>
      <td>{'lon': 20.866671, 'lat': -0.21667000000000003}</td>
      <td>218680</td>
      <td>70</td>
      <td>98</td>
      <td>92</td>
      <td>2.26</td>
      <td>0</td>
      <td>20</td>
    </tr>
    <tr>
      <th>121</th>
      <td>mersing</td>
      <td>my</td>
      <td>{'lon': 103.8405, 'lat': 2.4312}</td>
      <td>1732826</td>
      <td>77</td>
      <td>100</td>
      <td>8</td>
      <td>4.83</td>
      <td>2</td>
      <td>103</td>
    </tr>
    <tr>
      <th>174</th>
      <td>bolungarvik</td>
      <td>is</td>
      <td>{'lon': -23.261709, 'lat': 66.1517719}</td>
      <td>0</td>
      <td>39</td>
      <td>96</td>
      <td>68</td>
      <td>10.76</td>
      <td>66</td>
      <td>-23</td>
    </tr>
    <tr>
      <th>344</th>
      <td>nome</td>
      <td>us</td>
      <td>{'lon': -165.406387, 'lat': 64.501106}</td>
      <td>5870133</td>
      <td>42</td>
      <td>60</td>
      <td>20</td>
      <td>12.75</td>
      <td>64</td>
      <td>-165</td>
    </tr>
    <tr>
      <th>553</th>
      <td>hanamaki</td>
      <td>jp</td>
      <td>{'lon': 141.116669, 'lat': 39.383331}</td>
      <td>2112823</td>
      <td>57</td>
      <td>76</td>
      <td>75</td>
      <td>1.12</td>
      <td>39</td>
      <td>141</td>
    </tr>
    <tr>
      <th>220</th>
      <td>thompson</td>
      <td>ca</td>
      <td>{'lon': -97.855789, 'lat': 55.7435}</td>
      <td>6165406</td>
      <td>53</td>
      <td>66</td>
      <td>90</td>
      <td>3.36</td>
      <td>55</td>
      <td>-97</td>
    </tr>
    <tr>
      <th>739</th>
      <td>kokuy</td>
      <td>ru</td>
      <td>{'lon': 117.552902, 'lat': 52.203701}</td>
      <td>2021908</td>
      <td>52</td>
      <td>76</td>
      <td>88</td>
      <td>9.31</td>
      <td>52</td>
      <td>117</td>
    </tr>
    <tr>
      <th>636</th>
      <td>ahuimanu</td>
      <td>us</td>
      <td>{'lon': -157.837784, 'lat': 21.444719}</td>
      <td>5856516</td>
      <td>80</td>
      <td>78</td>
      <td>75</td>
      <td>14.99</td>
      <td>21</td>
      <td>-157</td>
    </tr>
    <tr>
      <th>728</th>
      <td>phu khieo</td>
      <td>th</td>
      <td>{'lon': 102.1287289, 'lat': 16.3763389}</td>
      <td>0</td>
      <td>78</td>
      <td>85</td>
      <td>88</td>
      <td>3.15</td>
      <td>16</td>
      <td>102</td>
    </tr>
    <tr>
      <th>431</th>
      <td>martapura</td>
      <td>id</td>
      <td>{'lon': 114.849998, 'lat': -3.41667}</td>
      <td>1636022</td>
      <td>78</td>
      <td>100</td>
      <td>44</td>
      <td>5.06</td>
      <td>-3</td>
      <td>114</td>
    </tr>
    <tr>
      <th>304</th>
      <td>freeport</td>
      <td>us</td>
      <td>{'lon': -89.621231, 'lat': 42.296692}</td>
      <td>4893171</td>
      <td>89</td>
      <td>45</td>
      <td>1</td>
      <td>6.93</td>
      <td>42</td>
      <td>-89</td>
    </tr>
    <tr>
      <th>630</th>
      <td>monrovia</td>
      <td>lr</td>
      <td>{'lon': -10.7969, 'lat': 6.30054}</td>
      <td>2274895</td>
      <td>77</td>
      <td>94</td>
      <td>75</td>
      <td>10.2</td>
      <td>6</td>
      <td>-10</td>
    </tr>
    <tr>
      <th>246</th>
      <td>esperance</td>
      <td>au</td>
      <td>{'lon': 121.900002, 'lat': -33.866669}</td>
      <td>2071860</td>
      <td>57</td>
      <td>79</td>
      <td>0</td>
      <td>16.24</td>
      <td>-33</td>
      <td>121</td>
    </tr>
    <tr>
      <th>623</th>
      <td>meadow lake</td>
      <td>ca</td>
      <td>{'lon': -108.434708, 'lat': 54.13348}</td>
      <td>6071421</td>
      <td>73</td>
      <td>38</td>
      <td>40</td>
      <td>14.99</td>
      <td>54</td>
      <td>-108</td>
    </tr>
    <tr>
      <th>631</th>
      <td>kayseri</td>
      <td>tr</td>
      <td>{'lon': 35.49683, 'lat': 38.73695}</td>
      <td>308463</td>
      <td>57</td>
      <td>47</td>
      <td>0</td>
      <td>9.17</td>
      <td>38</td>
      <td>35</td>
    </tr>
    <tr>
      <th>565</th>
      <td>aljezur</td>
      <td>pt</td>
      <td>{'lon': -8.8033, 'lat': 37.319149}</td>
      <td>2271968</td>
      <td>59</td>
      <td>100</td>
      <td>0</td>
      <td>6.06</td>
      <td>37</td>
      <td>-8</td>
    </tr>
    <tr>
      <th>581</th>
      <td>tanete</td>
      <td>id</td>
      <td>{'lon': 120.130402, 'lat': -5.337832}</td>
      <td>0</td>
      <td>75</td>
      <td>100</td>
      <td>88</td>
      <td>2.93</td>
      <td>-5</td>
      <td>120</td>
    </tr>
    <tr>
      <th>271</th>
      <td>cabo san lucas</td>
      <td>mx</td>
      <td>{'lon': -109.912376, 'lat': 22.890881}</td>
      <td>3985710</td>
      <td>95</td>
      <td>25</td>
      <td>5</td>
      <td>13.87</td>
      <td>22</td>
      <td>-109</td>
    </tr>
    <tr>
      <th>348</th>
      <td>camacha</td>
      <td>pt</td>
      <td>{'lon': -16.33333, 'lat': 33.083328}</td>
      <td>2270385</td>
      <td>60</td>
      <td>72</td>
      <td>75</td>
      <td>13.87</td>
      <td>33</td>
      <td>-16</td>
    </tr>
    <tr>
      <th>232</th>
      <td>marcona</td>
      <td>pe</td>
      <td>{'lon': -75.0844757, 'lat': -15.3439659}</td>
      <td>0</td>
      <td>60</td>
      <td>100</td>
      <td>56</td>
      <td>10.65</td>
      <td>-15</td>
      <td>-75</td>
    </tr>
    <tr>
      <th>558</th>
      <td>havelock</td>
      <td>us</td>
      <td>{'lon': -76.901329, 'lat': 34.879051}</td>
      <td>4470244</td>
      <td>80</td>
      <td>69</td>
      <td>5</td>
      <td>18.34</td>
      <td>34</td>
      <td>-76</td>
    </tr>
    <tr>
      <th>147</th>
      <td>yago</td>
      <td>mx</td>
      <td>{'lon': -105.066673, 'lat': 21.83333}</td>
      <td>3979983</td>
      <td>89</td>
      <td>49</td>
      <td>5</td>
      <td>6.93</td>
      <td>21</td>
      <td>-105</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>479</th>
      <td>alyangula</td>
      <td>au</td>
      <td>{'lon': 136.419174, 'lat': -13.84833}</td>
      <td>2079582</td>
      <td>80</td>
      <td>89</td>
      <td>48</td>
      <td>11.77</td>
      <td>-13</td>
      <td>136</td>
    </tr>
    <tr>
      <th>641</th>
      <td>salina cruz</td>
      <td>mx</td>
      <td>{'lon': -95.199997, 'lat': 16.16667}</td>
      <td>3520064</td>
      <td>87</td>
      <td>49</td>
      <td>56</td>
      <td>2.37</td>
      <td>16</td>
      <td>-95</td>
    </tr>
    <tr>
      <th>459</th>
      <td>kidal</td>
      <td>ml</td>
      <td>{'lon': 1.40778, 'lat': 18.441111}</td>
      <td>2455290</td>
      <td>85</td>
      <td>21</td>
      <td>24</td>
      <td>6.4</td>
      <td>18</td>
      <td>1</td>
    </tr>
    <tr>
      <th>351</th>
      <td>lebu</td>
      <td>cl</td>
      <td>{'lon': -73.650002, 'lat': -37.616669}</td>
      <td>3883457</td>
      <td>54</td>
      <td>100</td>
      <td>20</td>
      <td>0.69</td>
      <td>-37</td>
      <td>-73</td>
    </tr>
    <tr>
      <th>729</th>
      <td>oranjemund</td>
      <td>na</td>
      <td>{'lon': 16.433331, 'lat': -28.549999}</td>
      <td>3354071</td>
      <td>58</td>
      <td>100</td>
      <td>76</td>
      <td>11.21</td>
      <td>-28</td>
      <td>16</td>
    </tr>
    <tr>
      <th>412</th>
      <td>buala</td>
      <td>sb</td>
      <td>{'lon': 159.592117, 'lat': -8.14497}</td>
      <td>2109528</td>
      <td>83</td>
      <td>89</td>
      <td>92</td>
      <td>3.94</td>
      <td>-8</td>
      <td>159</td>
    </tr>
    <tr>
      <th>160</th>
      <td>hasaki</td>
      <td>jp</td>
      <td>{'lon': 140.833328, 'lat': 35.73333}</td>
      <td>2112802</td>
      <td>65</td>
      <td>82</td>
      <td>75</td>
      <td>5.82</td>
      <td>35</td>
      <td>140</td>
    </tr>
    <tr>
      <th>801</th>
      <td>nacala</td>
      <td>mz</td>
      <td>{'lon': 40.6854309, 'lat': -14.5656065}</td>
      <td>0</td>
      <td>78</td>
      <td>98</td>
      <td>0</td>
      <td>10.31</td>
      <td>-14</td>
      <td>40</td>
    </tr>
    <tr>
      <th>260</th>
      <td>huejuquilla el alto</td>
      <td>mx</td>
      <td>{'lon': -103.866669, 'lat': 22.6}</td>
      <td>4004616</td>
      <td>82</td>
      <td>20</td>
      <td>92</td>
      <td>2.71</td>
      <td>22</td>
      <td>-103</td>
    </tr>
    <tr>
      <th>733</th>
      <td>talnakh</td>
      <td>ru</td>
      <td>{'lon': 88.397202, 'lat': 69.486504}</td>
      <td>1490256</td>
      <td>28</td>
      <td>94</td>
      <td>44</td>
      <td>4.61</td>
      <td>69</td>
      <td>88</td>
    </tr>
    <tr>
      <th>510</th>
      <td>inyonga</td>
      <td>tz</td>
      <td>{'lon': 32.066669, 'lat': -6.71667}</td>
      <td>159134</td>
      <td>71</td>
      <td>54</td>
      <td>92</td>
      <td>6.51</td>
      <td>-6</td>
      <td>32</td>
    </tr>
    <tr>
      <th>484</th>
      <td>norman wells</td>
      <td>ca</td>
      <td>{'lon': -126.832901, 'lat': 65.282013}</td>
      <td>6089245</td>
      <td>48</td>
      <td>61</td>
      <td>90</td>
      <td>16.11</td>
      <td>65</td>
      <td>-126</td>
    </tr>
    <tr>
      <th>237</th>
      <td>batagay-alyta</td>
      <td>ru</td>
      <td>{'lon': 130.411392, 'lat': 67.80056}</td>
      <td>2027042</td>
      <td>46</td>
      <td>66</td>
      <td>0</td>
      <td>4.05</td>
      <td>67</td>
      <td>130</td>
    </tr>
    <tr>
      <th>133</th>
      <td>amambai</td>
      <td>br</td>
      <td>{'lon': -55.22583, 'lat': -23.10417}</td>
      <td>3472391</td>
      <td>68</td>
      <td>70</td>
      <td>0</td>
      <td>9.75</td>
      <td>-23</td>
      <td>-55</td>
    </tr>
    <tr>
      <th>272</th>
      <td>mazyr</td>
      <td>by</td>
      <td>{'lon': 29.2456, 'lat': 52.0495}</td>
      <td>625324</td>
      <td>52</td>
      <td>58</td>
      <td>8</td>
      <td>9.75</td>
      <td>52</td>
      <td>29</td>
    </tr>
    <tr>
      <th>593</th>
      <td>cherskiy</td>
      <td>ru</td>
      <td>{'lon': 161.300003, 'lat': 68.75}</td>
      <td>2126199</td>
      <td>42</td>
      <td>75</td>
      <td>12</td>
      <td>3.71</td>
      <td>68</td>
      <td>161</td>
    </tr>
    <tr>
      <th>107</th>
      <td>yellowknife</td>
      <td>ca</td>
      <td>{'lon': -114.352547, 'lat': 62.456001}</td>
      <td>6185377</td>
      <td>57</td>
      <td>54</td>
      <td>75</td>
      <td>10.29</td>
      <td>62</td>
      <td>-114</td>
    </tr>
    <tr>
      <th>247</th>
      <td>roblin</td>
      <td>ca</td>
      <td>{'lon': -77.016144, 'lat': 44.383419}</td>
      <td>6125860</td>
      <td>66</td>
      <td>88</td>
      <td>90</td>
      <td>13.87</td>
      <td>44</td>
      <td>-77</td>
    </tr>
    <tr>
      <th>706</th>
      <td>maamba</td>
      <td>zm</td>
      <td>{'lon': 27.15, 'lat': -17.366671}</td>
      <td>908718</td>
      <td>61</td>
      <td>91</td>
      <td>92</td>
      <td>2.71</td>
      <td>-17</td>
      <td>27</td>
    </tr>
    <tr>
      <th>27</th>
      <td>kapaa</td>
      <td>us</td>
      <td>{'lon': -159.318954, 'lat': 22.075211}</td>
      <td>5848280</td>
      <td>75</td>
      <td>94</td>
      <td>90</td>
      <td>18.34</td>
      <td>22</td>
      <td>-159</td>
    </tr>
    <tr>
      <th>94</th>
      <td>new norfolk</td>
      <td>au</td>
      <td>{'lon': 147.058701, 'lat': -42.782612}</td>
      <td>2155415</td>
      <td>41</td>
      <td>86</td>
      <td>75</td>
      <td>9.17</td>
      <td>-42</td>
      <td>147</td>
    </tr>
    <tr>
      <th>499</th>
      <td>terra santa</td>
      <td>br</td>
      <td>{'lon': -56.486938, 'lat': -2.10417}</td>
      <td>3386467</td>
      <td>78</td>
      <td>88</td>
      <td>8</td>
      <td>1.7</td>
      <td>-2</td>
      <td>-56</td>
    </tr>
    <tr>
      <th>742</th>
      <td>kieta</td>
      <td>pg</td>
      <td>{'lon': 155.633331, 'lat': -6.21667}</td>
      <td>2094027</td>
      <td>83</td>
      <td>97</td>
      <td>24</td>
      <td>4.72</td>
      <td>-6</td>
      <td>155</td>
    </tr>
    <tr>
      <th>331</th>
      <td>leningradskiy</td>
      <td>ru</td>
      <td>{'lon': 178.416672, 'lat': 69.383331}</td>
      <td>2123814</td>
      <td>27</td>
      <td>100</td>
      <td>44</td>
      <td>13.67</td>
      <td>69</td>
      <td>178</td>
    </tr>
    <tr>
      <th>525</th>
      <td>kabanjahe</td>
      <td>id</td>
      <td>{'lon': 98.490799, 'lat': 3.1001}</td>
      <td>1214965</td>
      <td>65</td>
      <td>96</td>
      <td>24</td>
      <td>1.92</td>
      <td>3</td>
      <td>98</td>
    </tr>
    <tr>
      <th>10</th>
      <td>qaanaaq</td>
      <td>gl</td>
      <td>{'lon': -69.363213, 'lat': 77.483971}</td>
      <td>3831208</td>
      <td>20</td>
      <td>88</td>
      <td>12</td>
      <td>1.59</td>
      <td>77</td>
      <td>-69</td>
    </tr>
    <tr>
      <th>198</th>
      <td>new london</td>
      <td>us</td>
      <td>{'lon': -72.099518, 'lat': 41.355652}</td>
      <td>4839416</td>
      <td>76</td>
      <td>83</td>
      <td>20</td>
      <td>3.36</td>
      <td>41</td>
      <td>-72</td>
    </tr>
    <tr>
      <th>360</th>
      <td>ahipara</td>
      <td>nz</td>
      <td>{'lon': 173.166672, 'lat': -35.166672}</td>
      <td>2194098</td>
      <td>61</td>
      <td>90</td>
      <td>92</td>
      <td>22.95</td>
      <td>-35</td>
      <td>173</td>
    </tr>
    <tr>
      <th>154</th>
      <td>puerto madryn</td>
      <td>ar</td>
      <td>{'lon': -65.038513, 'lat': -42.769199}</td>
      <td>3840092</td>
      <td>53</td>
      <td>77</td>
      <td>0</td>
      <td>10.54</td>
      <td>-42</td>
      <td>-65</td>
    </tr>
    <tr>
      <th>303</th>
      <td>shellbrook</td>
      <td>ca</td>
      <td>{'lon': -106.401093, 'lat': 53.216789}</td>
      <td>6145951</td>
      <td>69</td>
      <td>68</td>
      <td>90</td>
      <td>8.05</td>
      <td>53</td>
      <td>-106</td>
    </tr>
  </tbody>
</table>
<p>500 rows × 10 columns</p>
</div>




```python
clean_col = city_selection.columns[4:]
clean_col
for c in clean_col:
    city_selection[c] = pd.to_numeric(city_selection[c], errors = 'coerce')
    city_selection = city_selection[city_selection[c].isnull()== False]
    
len(city_selection)
```

    C:\Users\Joe\Anaconda3\lib\site-packages\ipykernel_launcher.py:4: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      after removing the cwd from sys.path.
    




    499




```python
city_selection.dtypes
```




    city            object
    country         object
    coord           object
    id               int64
    Temperature    float64
    Humidity         int64
    Cloudiness       int64
    Wind Speed     float64
    Latitude         int64
    Longitude        int64
    dtype: object




```python
#Temp vs Latitude Plot
plt.scatter(city_selection['Latitude'], city_selection['Temperature'], alpha = 0.5,)
plt.title("Temperature (F) vs. Latitude, Date Analysis 5.26.18 ")
plt.xlabel("Latitude")
plt.ylabel("Temperature (F)")
plt.xlim(-90, 90)
plt.show()
plt.savefig("Temp_V_Lat.png")

```


![png](Spiwak_WeatherPy_files/Spiwak_WeatherPy_22_0.png)



    <matplotlib.figure.Figure at 0x2952c167eb8>



```python
#Latitude V Humidity
plt.scatter(city_selection['Latitude'], city_selection['Humidity'], alpha = 0.5)
plt.title("Humidity vs. Latitude, Date Analysis 5.26.18 ")
plt.xlabel("Latitude")
plt.ylabel("Humidity")
plt.xlim(-90, 90)
plt.show()

plt.savefig("Humidity_V_Lat.png")


```


![png](Spiwak_WeatherPy_files/Spiwak_WeatherPy_23_0.png)



    <matplotlib.figure.Figure at 0x2952c79d908>



```python
#Cloudiness V Latitude
plt.scatter(city_selection['Latitude'], city_selection['Cloudiness'], alpha = 0.5)
plt.title("Cloudiness vs. Latitude, Date Analysis 5.26.18 ")
plt.xlabel("Latitude")
plt.ylabel("Cloudiness")
plt.xlim(-90, 90)
plt.show()

plt.savefig("Cloudiness_V_Lat.png")


```


![png](Spiwak_WeatherPy_files/Spiwak_WeatherPy_24_0.png)



    <matplotlib.figure.Figure at 0x2952c0fdc18>



```python
#Wind Speed V Latitude
plt.scatter(city_selection['Latitude'], city_selection['Wind Speed'], alpha = 0.5)
plt.title("Wind Speed vs. Latitude, Date Analysis 5.26.18 ")
plt.xlabel("Latitude")
plt.ylabel("Wind Speed")
plt.xlim(-90, 90)
plt.show()

plt.savefig("Wind_Speed_V_Lat.png")


```


![png](Spiwak_WeatherPy_files/Spiwak_WeatherPy_25_0.png)



    <matplotlib.figure.Figure at 0x2952c3a0eb8>



```python
#Mapping Out Cities by Lat + Lon, checking to make sure we are getting good global coverage
plt.scatter(city_selection['Longitude'], city_selection['Latitude'], alpha = 0.5)
plt.title(" Cities Analyzed, Date Analysis 5.26.18 ")
plt.show()

plt.savefig("Map.png")


```


![png](Spiwak_WeatherPy_files/Spiwak_WeatherPy_26_0.png)



    <matplotlib.figure.Figure at 0x2952c4a0978>


## Code running instruction 

### Initial_code.py
Initial_code.py contain the codes to download the results pages as .html file. Downloading the pages prevent being blocked by Tripadvisor though Tripadvisor doesn't seem to have any restriction on webscrapping. 

### Run_this_second.py
1. Run_this_second.py contain the codes to establish the connection to the Mongo database as well as create a new collection to store the data. In this stage, it will only scrape the info of the first search page downloaded in the previous stage. The purpose of downloading only the first page is that the first page contain the skeleton for the full database with no duplications of hotels. 
2. It will then add the info to the database created and create unique index on the hotel url to prevent the cases that hotel might have exactly the same names. Also, unique indexing on hotel url will also prevent duplications when updating the database.  
4. The codes used for the second stage will only scrape data from TripAdvisor with all the information available on the website like Address, Phone, Number of Reviews and Accessibility Scores. 

### Run_this_last.py 
1. At this stage, the code will update the database with the rest of pages we downloaded in the first stage (stage 2 only downloads the first page).
2. It will then update the database with prices info from Xotelo - an API that draw data directly from Tripadvisor as well as Agoda.com. 


## Snapshot of database displayed by Studio 3T
![alt text](https://github.com/tramnle/Webscrapping-TripAdvisor-MongoDB-/blob/main/MongoDB%20Tripadvisor%20Database.png?raw=true)



## Background

Emerged in the 1990s, online travel booking quickly bloomed into a big industry up until the present. It is now the major method for people to browse for travel accommodations like hotels, flights, or vehicle rentals. For OTA companies, they work as a third party that provides a full list of available accommodation that travelers can browse for. In most cases, the prices are identical between agencies. However, there are times that there is a mismatch between them which would attract more clients. In those cases, Travelocity acts as a useful tool to compare the price between the OTA. It provides not only the hotel information but also a compilation of hotel rates from multiple major booking platforms like Expedia, Booking.com, etc. Leveraging this structure, data engineers would be able to retrieve multiple prices using just Travelocity and use these data for analyzing purposes. For example, if a company can get the real-time price of a hotel in one city, it can adjust its own pricing accordingly. A company can also relate pricing and calculate price elasticity for different cities if hotel information is available.
In this project, we aim to develop a database structure that can be applied to retrieve hotel pricing of leading OTA companies that can update frequently. 


## Data Sources

We used Tripadvisor as a starting point to retrieve hotel URLs and unique hotel keys. We used this identity to match the same hotel in other OTA websites. We also considered retrieving hotel information of address, phone number, the total number of reviews, walkability, accessibility to restaurants, and attractiveness from this website.	
We consider several top OTA websites as hotel price data sources, including Bookings, Agoda, Hotels.com, and Expedia, each of which is a common choice people use to book hotels. 
And to avoid repetitive web scraping, we applied an open-source API, Xotelo.com, for direct retrieval of hotel prices. The API uses the key from Tripadvisor to request hotel prices listed in the popular OTA websites above. 
For simplicity, we only consider hotels in San Francisco that are available on Tripadvisor during 05/28/2022 and 05/30/2022, the Memorial Day weekends. However, the limitation of location and date can be easily changed or removed.	
Web-Scraping Routine
Our web-scraping routine is basically a two-step process.
The first step, as mentioned above, is to request unique hotel identities from Tripadvisor with the specified city name. In our case, it is San Francisco. So we concat ‘San Francisco’ with the search URL and send a ‘get’ request to Tripadvisor. We then parsed the response to get the hotel identities that are embedded in one of the CSS elements in HTML sources of the search result page. Using this information, we built the initial collection containing the Hotel Name, prices from 4 main OTA websites (Expedia, Booking.com, Hotels.com, and Agoda.com), and the Hotel URLs that link to the Tripadvisor page of the hotel. During the trial phase, we found that there are many duplications in the collection due to sponsored content and repetitive listing from Tripadvisor. To solve this problem, we built the initial collection using only the first search page and then created the unique index on the Hotel URL. Implementing this has helped us not insert documents with the same hotel link over again.
During the first phase, we faced a problem that the API can only give direct quotes for hotels that are listed as hotels in the major travel agencies. For the smaller one that is privately managed, the API is less likely to provide information on those hotels. Whenever it happens, we prefer to go back to Tripadvisor to get the information. By integrating Tripadvisor and Xotelo, it cuts back a significant amount of time for those hotels that are available in Xotelo but not on Tripadvisor. 
For the second phase, we will then refer back to Tripadvisor to scrape the other information like Address, Phone number, Number of reviews, and Location Scores. The process took a significantly shorter amount of time since, in this step, the length of stay and the check-in check-out dates are not specified. Only needed information listed above is retrieved from the page. 
The two-stage process is designed with consideration of weak coupling, so each one can run separately in case the other process breaks down. Try-except structure is also applied to avoid full breakdown of the program and alerting on time. Also, the code structure is designed to prevent crashing and blocking from Tripadvisor when running the code. After running the second source code (Run-this-second.py) which retrieves the info for the first search page, we can run the third source code any time and stop anywhere between the running process without replicating any of the documents in the collection since the unique index ensures the duplication problem will not happen. Besides, the city of interest and date of interest can be changed easily to allow extension for broader usage by editing the URL input inside of the search results page and the URL to request API from Xotelo. 
Database Design
We stored processed data in MongoDB, a cross-platform document-oriented database program. Using Python as the single coding language, we link to MongoDB directly from the Python package pymongo, which allows CRUD operations with Python objects. 
In the database, it currently contains only one collection ‘hotels_prices’. Each object inside of the collection contains Hotel Name, Expedia Rate, Bookings.com Rate, Hotels.com Rates, Agoda.com Rate, Hotel URL, Phone number, Number of Reviews, Greate for walker Score, Restaurant Scores, and Attraction Scores. Each attribute has the exact meaning of its name. 
For the attributes that contain numbers, only information retrieved from Xotelo is formatted as integer due to the format of the JSON objects of the API. Any other information retrieved directly from Tripadvisor is string only as most of the text embedded inside of the website is string formatted. 
For any information that is not available on both websites, it will either be left blank or as null values. Due to the nature of the API, prices can be not available from all 4 major agencies we were looking for so Null values appeared most often in the Rates attributes. For the Phone attribute, Tripadvisor does not always provide phone numbers for the hotel so missing values are also frequent for this attribute. For any other attributes, the information is mostly fully available. 

## Business Implications

As discussed above, the program has the ability to extend to wider use. And with the designed program, we used JSON-format files to store data, which can be easily accessed through Python and any programming language. Using Python, we can answer business questions like: for hotels with more than 500 hundred reviews, do they have a higher price in Agoda than Booking on average? Where are the cheapest and most expensive hotels located in San Francisco?  We can even extend the research to understand the relationship of attractiveness and hotel names, or make visualizations with geoinformation for decision-makers (Figure 2).
To be specific, the database can be used in Price-Demand analysis in hotel industries. Users of the data can use the data as price records for the specific time frame (Memorial Day) and combine with the number of bookings in some specific hotels as the demand. Besides, the database can be used as an instrument for hotel managers to set rates for the property after conducting some price sensitivity analysis. 
In terms of the user’s perspective, any person can use it as a source of price information to pick on where to book the hotel based on their price’s preference, location’s preference. This database would allow them to easily compare between hotels by looking at specific attributes that they desire. 
           NoSQL databases are generally more flexible than SQL databases. And MongoDB has the advantage of high performance, high availability, and automatic scaling. For instance, MongoDB does not require a primary key so any null values in hotel information can be handled, even the names or URLs; the address information is a combination of string, special characters, and integers, which can also be handled by the flexible datatype of MongoDB. And when the scale of information retrieved increases, MongoDB can automatically scale and even migrate into a distributed database, using the map-reduce procedure to retrieve data.
           
## Summary 
In this project, we developed an extendable program for scrapping hotel prices, locations, URLs, and other information from leading OTA websites in Python. We built the data pipeline of scraping and storing the retrieved data to NoSQL database MongoDB. 
The program can be utilized by companies in the OTA industry to track competitors’ pricing strategies, analyze hotel prices for different cities, etc., to help the companies to design their own pricing strategies.





import requests
from bs4 import BeautifulSoup
import pandas as pd

products = []

for page in range(1, 5):

    url = f"https://books.toscrape.com/catalogue/page-{page}.html"
    
    response = requests.get(url)

    soup = BeautifulSoup(response.text, "html.parser")

    books = soup.find_all("article", class_="product_pod")

    for book in books:

        name = book.h3.a["title"]

        price = book.find("p", class_="price_color").text

        rating = book.find("p")["class"][1]

        products.append([name, price, rating])

df = pd.DataFrame(products, columns=["Name", "Price", "Rating"])

print(df.head())

df.to_csv("products.csv", index=False)






LOAD DATA INPATH '/user/hive/flights_data.csv'
INTO TABLE flight_info_main; CREATE INDEX idx_airline
ON TABLE flight_info_main_new (airline)
AS 'COMPACT'
WITH DEFERRED REBUILD;


CREATE DATABASE flight_db;
USE flight_db;

CREATE TABLE flights (
    flight_id STRING,
    airline STRING,
    source STRING,
    destination STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';

CREATE TABLE bookings (
    booking_id STRING,
    flight_id STRING,
    passenger_name STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';

LOAD DATA LOCAL INPATH '/home/user/flights.txt' INTO TABLE flights;
LOAD DATA LOCAL INPATH '/home/user/bookings.txt' INTO TABLE bookings;

CREATE TABLE flights_hbase_hive (
    flight_id STRING,
    airline STRING,
    source STRING,
    destination STRING
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES (
    "hbase.columns.mapping" = ":key,info:airline,info:source,info:destination"
)
TBLPROPERTIES ("hbase.table.name" = "flights_hbase");

INSERT INTO TABLE flights_hbase_hive
SELECT flight_id, airline, source, destination
FROM flights;

SELECT 
    b.booking_id,
    b.passenger_name,
    f.airline,
    f.source,
    f.destination
FROM bookings b
INNER JOIN flights f
ON b.flight_id = f.flight_id;

SELECT 
    b.booking_id,
    b.passenger_name,
    f.airline,
    f.source,
    f.destination
FROM bookings b
LEFT JOIN flights f
ON b.flight_id = f.flight_id;

SELECT 
    b.booking_id,
    b.passenger_name,
    f.flight_id,
    f.airline,
    f.source,
    f.destination
FROM bookings b
RIGHT JOIN flights f
ON b.flight_id = f.flight_id;

SELECT 
    b.booking_id,
    b.passenger_name,
    f.flight_id,
    f.airline,
    f.source,
    f.destination
FROM bookings b
FULL OUTER JOIN flights f
ON b.flight_id = f.flight_id;
ORDER BY flight_date;


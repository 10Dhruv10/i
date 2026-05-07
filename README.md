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

CREATE DATABASE IF NOT EXISTS flight_db;
USE flight_db;

DROP TABLE IF EXISTS flight_info_main;
DROP TABLE IF EXISTS airlines;
DROP TABLE IF EXISTS ext_flights;

CREATE TABLE flight_info_main (
    flight_id INT,
    airline STRING,
    origin STRING,
    destination STRING,
    departure_time STRING,
    arrival_time STRING,
    departure_delay INT,
    arrival_delay INT,
    flight_date STRING,
    status STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

CREATE TABLE airlines (
    airline_id STRING,
    airline_name STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

CREATE EXTERNAL TABLE ext_flights (
    flight_id INT,
    airline STRING,
    origin STRING,
    destination STRING,
    departure_time STRING,
    arrival_time STRING,
    departure_delay INT,
    arrival_delay INT,
    flight_date STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/user/hive/external/flights/';

LOAD DATA INPATH '/user/hive/flights_data.csv'
INTO TABLE flight_info_main;

INSERT INTO flight_info_main VALUES
(101, 'AA', 'JFK', 'LAX', '10:00', '13:00', 15, 10, '2008-01-01', 'On Time');

INSERT INTO flight_info_main
SELECT * FROM ext_flights;

ALTER TABLE flight_info_main ADD COLUMNS (remarks STRING);

ALTER TABLE flight_info_main RENAME TO flight_info_main_new;

SELECT 
    f.flight_id,
    f.airline,
    a.airline_name,
    f.origin,
    f.destination
FROM flight_info_main_new f
JOIN airlines a
ON f.airline = a.airline_id;

CREATE INDEX idx_airline
ON TABLE flight_info_main_new (airline)
AS 'COMPACT'
WITH DEFERRED REBUILD;

ALTER INDEX idx_airline ON flight_info_main_new REBUILD;

SELECT 
    flight_date,
    AVG(departure_delay) AS avg_departure_delay
FROM flight_info_main_new
WHERE flight_date LIKE '2008%'
GROUP BY flight_date
ORDER BY flight_date;


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

# newegg_scraping
from bs4 import BeautifulSoup
import requests
import re
from csv import writer


search_term = input("What product do you want to search for? ")

url = f"https://www.newegg.com/global/il-en/p/pl?d={search_term}&N=4131"
page = requests.get(url).text
doc = BeautifulSoup(page, "html.parser")

page_text = doc.find(class_="list-tool-pagination-text").strong


pages = int(str(page_text).split("/")[-2].split(">")[-1][:-1])

items_found = {}

with open('serching.csv', 'w') as f:
    thewriter = writer(f)
    header = ['Item', 'Price', 'Link']
    thewriter.writerow(header)

    for page in range(1, pages + 1):
        url = f"https://www.newegg.ca/p/pl?d={search_term}&N=4131&page={page}"
        page = requests.get(url).text
        doc = BeautifulSoup(page, "html.parser")

        div = doc.find(
            class_="item-cells-wrap border-cells items-grid-view four-cells expulsion-one-cell")
        items = div.find_all(text=re.compile(search_term))

        for item in items:
            parent = item.parent
            if parent.name != "a":
                continue

            link = parent['href']
            next_parent = item.find_parent(class_="item-container")
            try:
                price = next_parent.find(
                    class_="price-current").find("strong").string
                items_found[item] = {"price": int(
                    price.replace(",", "")), "link": link}
            except:
                pass

    sorted_items = sorted(items_found.items(), key=lambda x: x[1]['price'])

    for item in sorted_items:

        info = [item[0], f"${item[1]['price']}", item[1]['link']]
        thewriter.writerow(info)

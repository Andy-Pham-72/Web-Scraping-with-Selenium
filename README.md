# Web Scraping with Selenium.

In the other [repository](https://github.com/Andy-Pham-72/Web-Scraping-with-BeautifulSoup-and-Pandas), I introduced how to use Beautiful Soup to do a variety of quick and simple web-scraping tasks. Beautiful Soup is great, but it has its limitations, because sometimes the content we want to scrape is hidden behind buttons and links that we cannot access directly through a URL. There are also a variety of more sophisticated browsing tasks, like filling in forms and text boxes, that we might have the need to automate. For these purposes it is good to reach for another package: `selenium`

[Selenium](https://www.selenium.dev/) is actually much more than a Python package; it's a whole framework for automating web browsers for the purposes of testing web applications, and it's been ported to a variety of programming languages in addition to Python. The main reason you should be aware of this is because, if you ever need to Google something about Selenium, you should include Selenium *and* Python in your search query; otherwise you will probably get a lot of results in Java (and who wants that). Also, if you ever have any questions about Selenium, their [unofficial documentation](https://selenium-python.readthedocs.io/index.html) is always a good place to start.

---
### Install Selenium
Open bash and run:
```bash
pip install selenium
```
---


```python
# Standard imports
import pandas as pd

# For web scraping
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions
from selenium.webdriver.common.by import By
import time
```

### Download driver

In order to use Selenium, **you must download a driver to interface with your chosen browser**. Currently, Selenium supports Chrome, Firefox, Safari, and Edge. You can find a link to the browser of your choice [here](https://selenium-python.readthedocs.io/installation.html#drivers). **Be sure to download the driver that matches the version of your chosen browser!**

For the purposes of this kickoff, we'll be using Selenium with Chrome. To do this, we must first [download ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/downloads). For example, if your current Chrome browser version is 89, you have to download Chromedriver version 89.

The ChromeDriver file, once unzipped, is a single executable file called `chromedriver`. You may keep this file anywhere on your computer, but it is best to place it in an easy-to-reference location where you know. For example, you save `chromedriver` in the `Download` folder, you have to get the directory path as `/Users/Download/chromedriver` . Thus, we can assign that directory path to a variable:


```python
# Save path to chromedriver executable file to variable
chrome_path = '/Users/quanganhpham/Downloads/chromedriver' # for example, /Users/Download/chromedriver
#Directory path to your chromedriver
```

**Today we will be scraping the product `Solaray, Vitamin D3 + K2, Soy-Free, 125 mcg (5000 IU), 60 VegCaps` information (names / brand / review content) from this [LINK](https://ca.iherb.com/pr/Solaray-Vitamin-D3-K2-Soy-Free-125-mcg-5000-IU-60-VegCaps/70098).**


```python
# Initializes the Chrome Driver to access the website
driver = webdriver.Chrome(chrome_path)
driver1 = webdriver.Chrome(chrome_path)


# Assigns url into a variable
product_url = "https://ca.iherb.com/pr/Solaray-Vitamin-D3-K2-Soy-Free-125-mcg-5000-IU-60-VegCaps/70098"

# Initializes the Chrome Driver to access the URL
driver.get(product_url)

```

First, we can get the product brand in the main product page, by using this syntax: `find_element_by_xpath('.//*[@id="brand"]/a/span/bdi').get_attribute('textContent')` which means it will find the element with the XPATH starting from `id='brand'` to the order of following tags `a` -> `span` -> `bdi` ; then we can use the funtion `get_attribute()` with the `textContent` to parse the product brand which is `Solaray`.

(We can also parse the product name from the main page; However, we won't do it because we want to try another function `find_element_by_css_selector` later)

<br>
<br>
<img src = "https://docs.google.com/uc?export=download&id=1HN4TWV3-vjPNTcoXDlBAWyUumhaqpJ2i" />

<br>
<br>


We want to scrape the review contents of the product. However, the all the review contents are located in this [View All Reviews](https://ca.iherb.com/r/Solaray-Vitamin-D3-K2-Soy-Free-125-mcg-5000-IU-60-VegCaps/70098) instead of the main product page url. Therefore, we have to get that link by using the Seleninum functions to parse the attribute which contains the url to all the reviews as in the below picture: 


<br>
<br>
<img src = "https://docs.google.com/uc?export=download&id=14gKg45a0oUJ-5rQwFnB-wNGixYDcu8Hv" />

<br>
<br>

And for the product name, we can find it in the [View All Reviews](https://ca.iherb.com/r/Solaray-Vitamin-D3-K2-Soy-Free-125-mcg-5000-IU-60-VegCaps/70098?p=1&sort=6). This is when this function `find_element_by_css_selector` comes in handy! using the following syntax: `find_element_by_css_selector('[class="nav-product-link-text"] span').text)` which means it can find the element starting with `class="nav-product-link-text"` -> `span` tag -> extract the text from that tag with `.text` as in the below picture:


<br>
<br>
<img src = "https://docs.google.com/uc?export=download&id=1SaN-X75OfH1zF_R9DQiwqugJ47obgJYt" />

<br>
<br>



```python
# Set a waiting time for the Driver
wait = WebDriverWait(driver, 4)

# Locate `View All Reviews` link
link = wait.until(expected_conditions.presence_of_element_located((By.CSS_SELECTOR,"span.all-reviews-link > a")))

# Get `View All Reviews` link
x = link.get_attribute("href")

# Check the link
x
```




    'https://ca.iherb.com/r/Solaray-Vitamin-D3-K2-Soy-Free-60-VegCaps/70098'



**Now we have to create 2 for loops:** <br>
    - In the first loop, we will get the link of different review pages which we want to scrape.<br>
    - In the second loop, we will scrape the data that we need.


```python
# Create lists for the dataframe:
item_name = []
item_brand = []
review_contents = []

# Scrape maximum 3 pages in the review section
max_page_num = 3

for page_num in range(1, max_page_num + 1):
    
    review_url = x + "?&p=" + str(page_num)
    print(review_url)
    
    # Initializes the Chrome Driver to access review_url
    driver1.get(review_url)
    
    # Get the all the review elements
    review_containers = driver1.find_elements_by_class_name('review-row')
    
    for container in review_containers:
        # Add the review contents
        review_contents.append(container.find_element_by_class_name('review-text').text)
        # Add the product name
        item_name.append(driver1.find_element_by_css_selector('[class="nav-product-link-text"] span').text)
        # Add the product brand
        item_brand.append(driver.find_element_by_xpath('.//*[@id="brand"]/a/span/bdi').get_attribute('textContent'))
    
    # Sleep
    time.sleep(4) 
```

    https://ca.iherb.com/r/Solaray-Vitamin-D3-K2-Soy-Free-60-VegCaps/70098?&p=1
    https://ca.iherb.com/r/Solaray-Vitamin-D3-K2-Soy-Free-60-VegCaps/70098?&p=2
    https://ca.iherb.com/r/Solaray-Vitamin-D3-K2-Soy-Free-60-VegCaps/70098?&p=3



```python
# Create a dataframe
df_product = pd.DataFrame({'item_brand'   : item_brand, 
                            'item_name'   : item_name, 
                        'review_contents' : review_contents }) 

# Check the dataframe shape
df_product.shape
```




    (20, 3)




```python
# Check the dataframe
df_product.head(15)
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
      <th>item_brand</th>
      <th>item_name</th>
      <th>review_contents</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>Everyone around said that in Russia everyone, ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>So far I can not appreciate the dignity of thi...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>I am surprised by the reviews of people who de...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>very cool product, I</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>Very cool product, I recommend it to everyone</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>cool very cool product recommend it</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>I recommend</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>Because of the large cans, they noticed that t...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>Very, very cool product, I recommend</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>After a course of these vitamins, as my nutrit...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>I love that this supplement contains vitamin K...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>The excellent formula of this drug will provid...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>Simply the best vitamin D3 complex! The dosage...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>After reading reviews about the lack of vitami...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Solaray</td>
      <td>Vitamin D3 + K2, Soy-Free, 60 VegCaps</td>
      <td>They drank the whole family. Raises vitamin D ...</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Let make a CSV file from the dataframe
df_product.to_csv ('product_review.csv', index = False, header=True) 

```

Lastly, you can also use Selenium to close your browser. (Or you just need to simply close the browser.)


```python
driver.quit()
driver1.quit()
```

**END**

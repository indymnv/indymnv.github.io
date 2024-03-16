---
author: Navi
title: How to scrape data with Python using selenium and Pandas
date: 2022-12-15
Lastmod: 2024-03-16
description: A simple example of scraping using selenium
categories:
  - programming
tags:
  - python
  - data science
  - web scraping
---

## Introduction
In this tutorial, I will dedicate myself to explaining how web scraping can be done from a platform where a dynamic interaction of the web application is required, this is quite useful when obtaining data from different links within the platform and where it is necessary a management scheme of the front-end components to carry it out.

Here there are mainly two essential libraries, the first is selenium which corresponds to a framework that operates for multiple languages and serves to automate and control the browser, while Pandas for data manipulation will allow us to read data tables directly.

Many times, the beautiful soup library is used to extract html elements from the web, but as we will see, it is not necessary to do so in this case.

For this example, I am going to use the [Chilean dairy production platform](http://aplicativos.odepa.cl/recepcion-industria-lactea.do), this platform is used to obtain information on the production of products dairy products from different factories nationwide.

## Requirements
To start, you must have Python installed. In my case, I am using version 3.9, you also have to have your browser (Mozilla or Chrome) secured. In this project, I will use the chrome one, but the codes should be similar to the one we are using here, then to work with selenium, you have to download the [executable](https://chromedriver.chromium.org/downloads) that corresponds to your browser and its respective version

If you use pip you can install it using:

```python
pip install -U selenium
```
Then import the libraries.

```python

from selenium import webdriver
import pandas as pd
import lxml
from selenium.webdriver.support.ui import Select
import sys
import time

```
Once you have imported the corresponding libraries, we will perform the first test with the chromedriver.exe (the one you downloaded from the selenium portal). For simplicity, I recommend having it in the same directory as this scrapper.

```python

driver = webdriver.Chrome('/Your/path/to/the/project/chromedriver')
driver.get("http://aplicativos.odepa.cl/recepcion-industria-lactea.do")

```
This should allow the web page to be opened from the Chrome browser, the driver variable that we have assigned the chromedriver will drive the states of our browser. We can now add this snippet code

```python

time.sleep(5)
driver.quit()

```
With this, we add a timeout of 5 seconds, and with driver.quit() we close the browser. The reason for adding waiting times is that while we have to operate within the browser, either due to internet connections or latency of the web platform, we will therefore have to wait for the elements we need to be available.

It is time to see how we can start interacting with the web page elements. For example, if we want to click on certain features, what we have to do is right-click on the component on the web page, place inspect and then recognize the element and how we can call it according to how it is identified, this can be by id, name, XPath, etc. I often use the XPath, which you can copy and paste into your code.

```python

#Select elements
driver.find_element_by_id('tipoConsulta2').click()
driver.find_element_by_id('filterByRegionOrPlanta2').click()
driver.find_element_by_id('filterByRegionOrPlanta2').click()

#Extract the list of years
driver.find_element_by_xpath('//*[@id="divFechaDetalleMensual"]/img').click()
driver.find_element_by_xpath('//*[@id="ui-datepicker-div"]/div[1]/div/select').click()
years = driver.find_elements_by_tag_name("option")

```
Here what we have done is open the web page and make the necessary selections and filters to access the data, we end up creating a list called years, where we will have all the years available in this web application.

Now with this, we can get the elements. Using the following code.

```python

list_years = []
for year in years:
    list_years.append(year.get_attribute('value'))

#here I added a filter by year which is optional (you can delete it)
list_years = [element for element in list_years if element != '' and int(element)> 2000]

```
Now we will obtain the list of elements of all the years to be able to iterate. Then if we want to get the plants, we can use the following:

```python

#Extract all the factory names:
plantasposibles=driver.find_element_by_id('planta')
plantasposibles=plantasposibles.find_elements_by_tag_name("option")
valoresplantas=[]
nombresplantas=[]

for option in plantasposibles:
    valoresplantas.append(option.get_attribute("value"))
    nombresplantas.append(option.get_attribute("text"))

```

We locate the dropdown that corresponds to the list of available plants, with this, we take the elements and build the list of plants. This will allow us to perform the following iteration:

```python

tabla=pd.DataFrame() #Here we create the dataframe

driver.find_element_by_xpath('//*[@id="divFechaDetalleMensual"]/img').click()

for lastyear in list_years:
    for i in range(1,len(valoresplantas)):
    ...

```
We need to start controlling the options and release the report with the data. From there, we perform reading and data extraction, this is where Pandas shines. If we remember the last double loop, what should go inside is the following.

```python

#Select options
driver.execute_script("document.getElementById('planta').value="+ valoresplantas[i])
driver.find_element_by_xpath("//*[@id='divFechaDetalleMensual']/img").click()
time.sleep(1)
select=Select(driver.find_element_by_xpath("//*[@id='ui-datepicker-div']/div[1]/div/select"))
select.select_by_visible_text(str(lastyear))        
driver.find_element_by_xpath("//*[@id='ui-datepicker-div']/div[2]/button").click()
driver.find_element_by_id('fechaDetalleMensual').send_keys(lastyear)
timeout=15
driver.find_element_by_id('btnVerInforme').click()
timeout=20

############################## PANDAS #######################################

prueba_html=driver.page_source
df = pd.read_html(prueba_html, flavor='html5lib')[0]
df=df.drop(df.columns[14:397],axis=1)
df=df.drop(df.index[0:8],axis=0)
df=df.drop(df.index[1],axis=0)
df=df.drop(df.index[8:9],axis=0)
df['Year']=lastyear
df['Factory_Name']=nombresplantas[i]
tabla=pd.concat([tabla,df])

```
In case it fails, which is typical when working in selenium, the try/catch options are the best to handle exceptions intelligently. Obviously, it depends a lot on the case and the nature of the project on how to use them, but here I just proceeded to close the application and operate again where it was. To summarize this point, the double loop would look like this:

```python

for lastyear in list_years:
    for i in range(1,len(valoresplantas)):
        try: 
            driver.execute_script("document.getElementById('planta').value="+ valoresplantas[i])
            driver.find_element_by_xpath("//*[@id='divFechaDetalleMensual']/img").click()
            time.sleep(1)
            select=Select(driver.find_element_by_xpath("//*[@id='ui-datepicker-div']/div[1]/div/select"))
            select.select_by_visible_text(str(lastyear))        
            driver.find_element_by_xpath("//*[@id='ui-datepicker-div']/div[2]/button").click()
            driver.find_element_by_id('fechaDetalleMensual').send_keys(lastyear)
            timeout=15
            driver.find_element_by_id('btnVerInforme').click()
            timeout=20
            
            
            prueba_html=driver.page_source
            df = pd.read_html(prueba_html, flavor='html5lib')[0]
            df=df.drop(df.columns[14:397],axis=1)
            df=df.drop(df.index[0:8],axis=0)
            df=df.drop(df.index[1],axis=0)
            df=df.drop(df.index[8:9],axis=0)
            df['Year']=lastyear
            df['Factory_Name']=nombresplantas[i]
            tabla=pd.concat([tabla,df])

        except:

            #If fail close and open up the window again
            #driver.quit()
            time.sleep(5)
            driver.get("http://aplicativos.odepa.cl/recepcion-industria-lactea.do")
            time.sleep(5)
            driver.find_element_by_id('tipoConsulta2').click()
            driver.find_element_by_id('filterByRegionOrPlanta2').click()
            driver.find_element_by_id('filterByRegionOrPlanta2').click()

```

## Final steps with Pandas

With this, we would be finishing the process, the only thing left is to integrate the final data with some extra elements and save the dataframe. We will ensure that each component is integrated with its period since the months are in columns, so we will make a single column that contains them.

```python

tabla=tabla[['Year', 'Factory_Name', 'Product', 'Unit','Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec']]

lista=range(len(tabla.index))
tabla.index=lista

tablafinal=pd.DataFrame()
tablaparcial=tabla.drop(tabla.columns[4:],axis=1)

for month in tabla.columns[4:len(tabla.columns)]:

    tablaparcial['Month']=month
    tablaparcial['Quantity']=tabla[month]
    tablafinal=pd.concat([tablafinal,tablaparcial])

tablafinal.to_csv("data.csv", index = False)

```

Finally, we can do the extraction and a simple preprocessing to leave them more prepared for some analysis or save them to a database.

## Conclusions

In this project, we show how we can perform scrapping using selenium and pandas, this of course, can be done thanks to the pandas tools to extract data from HTML, simplifying the extraction. Selenium is an excellent tool to carry out this automation and test web pages, so I recommend it for the design of web apps, for example,  failures in the results or scenarios where there are possible bugs.




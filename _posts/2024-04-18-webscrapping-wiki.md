---
layout: post
title: "Scrapped data from Wikipedia"
categories: [Python, Code, Project]
---

Project information: 
+ Language: Python
+ Web application: Jupyter Notebook
+ Data: Covid-19
+ Objective: To scrap specific data from Wikipedia.


References: [AlexTheAnalyst](https://www.youtube.com/watch?v=qfyynHBFOsM&list=PLUaB-1hjhk8H48Pj32z4GZgGWyylqv85f)

1) Import libraries
```python
from bs4 import BeautifulSoup
import requests
```

2) Set the URL and assign the page (including texts) as 'soup'
```python
url = 'https://en.wikipedia.org/wiki/List_of_largest_companies_in_the_United_States_by_revenue'
page = requests.get(url)
soup = BeautifulSoup(page.text, 'html')
```

3) Print 'soup'
```python
print(soup)
```
Snippet of the result:

![image](https://github.com/user-attachments/assets/99647095-d637-4fe7-833d-cec5c822e57d)

4) Find the tag 'table'
```python
soup.find('table')
```
Snippet of the result:

![image](https://github.com/user-attachments/assets/a98e2779-c76d-4957-9313-2861c13113ce)

5) Find all tags for 'table'
```python
soup.find_all('table')[1]
```
Snippet of the result:

![image](https://github.com/user-attachments/assets/7f301b19-9070-4ec8-af14-6f75deca53d8)

6) Find the tags for 'table' with class named 'wikitable sortable'
```python
soup.find('table', class_= 'wikitable sortable')
```
Snippet of the result:

![image](https://github.com/user-attachments/assets/7447da80-7b3c-43c5-bd11-aac43fd7bf66)

7) Assign #5 to 'table' variable and then print the variable.
```python
table = soup.find_all('table')[1]
print(table)
```
Snippet of the result:

![image](https://github.com/user-attachments/assets/e9a8e56f-22d7-4550-a32b-3d4bc076d05c)

8) Find the table with 'th' tag, assign it to a variable called 'world_titles' and print the variable.
```sql
world_titles = table.find_all('th')
print(world_titles)
```
Snippet of the result:

![image](https://github.com/user-attachments/assets/29882ac1-5ad6-4ec7-b7a8-e8cec7147001)

9) Strip the text to to get the title and assign it to a variable called 'world_table_titles'.
```python
world_table_titles = [title.text.strip() for title in world_titles]
print(world_table_titles)
```
Snippet of the result:

![image](https://github.com/user-attachments/assets/a2405ba4-8cb7-4a7b-8b45-e31e9af0f90e)

10) Import library called pandas as pd.
```python
import pandas as pd
```

11) Assign a dataframe with the columns from 'world_table_titles'.
```python
df = pd.DataFrame(columns = world_table_titles)
df
```
Snippet of the result:

![image](https://github.com/user-attachments/assets/d2393446-2aaa-4967-b5bd-46f2ad9ea7d2)

12) In the column, find all 'tr' tags.
```python
column_data = table.find_all('tr')
```

13) Create a for loop to get the row data from 'td' tags and print the length
```python
for row in column_data[1:]:
    row_data = row.find_all('td')
    ind_row_data = [data.text.strip() for data in row_data]
    #print(ind_row_data)
    
    length = len(df)
    df.loc[length] = ind_row_data
    
df 
```
Snippet of the result:

![image](https://github.com/user-attachments/assets/32c7f796-f7b5-404f-9856-ccadb18bc3e8)

14) Save the scrapped data into a CSV file.
```python
df.to_csv(r'C:\Users\user\Documents\Data Analyst\web scrapping\WebScrappedCompanies.csv', index = False)
```

Full code and results can be found here [Github](https://github.com/imanazmi/PortfolioProjects/blob/main/Scrapping%20data%20from%20Wikipedia.ipynb)




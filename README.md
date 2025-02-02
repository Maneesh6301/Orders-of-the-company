# Orders-of-the-company
Performance of the company based on orders 
Downloaded the kaggle dataset




python code using pandas libaray
#!/usr/bin/env python
# coding: utf-8

# In[2]:


import kaggle


# In[3]:


get_ipython().system('kaggle datasets download ankitbansal06/retail-orders -f orders.csv')


# In[4]:


import zipfile
zip_ref = zipfile.ZipFile("orders.csv.zip") 
zip_ref.extractall() # extract file to dir
zip_ref.close() # close file


# In[5]:


import pandas as pd
import pandas.io
from pandas.io import sql
import pymysql


# In[6]:


df=pd.read_csv("orders.csv",na_values=["Not Available","unknown"])


# In[7]:


df.head()


# In[8]:


df.head(20)


# In[9]:


df["Ship Mode"].unique()


# In[10]:


df.columns=df.columns.str.lower()


# In[11]:


df.head()


# In[12]:


df.columns=df.columns.str.replace(" ","_")


# In[13]:


df.head()


# In[14]:


df["discount"]=df["list_price"]*df["discount_percent"]/100


# In[15]:


df.head()


# In[16]:


df


# In[17]:


df["sale_price"]=df["list_price"]-df["discount"]


# In[18]:


df


# In[19]:


df["profit"]=df["sale_price"]-df["cost_price"]


# In[20]:


df


# In[21]:


df.dtypes


# In[22]:


df["order_date"]=pd.to_datetime(df["order_date"],format="%Y-%m-%d")


# In[23]:


df.dtypes


# In[24]:


df=df.drop(columns=["cost_price"])


# In[25]:


df


# In[26]:


from sqlalchemy import create_engine
engine=create_engine("mysql+pymysql://root:123@127.0.0.1:3306/db")
conn=engine.connect()


# In[27]:


df.to_sql("orders",engine,index=False,if_exists="replace")
conn.close()









sql code 
#TOP 10 selling products
select  product_id,sum(sale_price) as sales
from orders
group by product_id
order by sales desc
limit 10

#comparison of sales between 2023 and 2022 with month
with xyz as(
SELECT MONTH(order_date) as mn ,year(order_date) as yr,sum(sale_price) as sales
from orders 
group by month(order_date),year(order_date)
)
select mn,
sum(case when yr=2023 then sales else 0 end) as sales_2023,
sum(case when yr=2022 then sales else 0 end ) as sales_2022
from xyz
group by mn
order by mn

#most selling category with month and year
#with xyz as(
select category as cat,month(order_date) as mn, year(order_date)as yr,sum(sale_price) as sales,
 ROW_NUMBER() OVER(partition by category ORDER  by sum(sale_price) desc) as ranked
from orders
group by category,month(order_date),year(order_date)
)
select * 
from xyz
where ranked=1


#percentage of profit comparision  between  2023 and 2022
with xyz as(
select sub_category as sc,year(order_date) as yr,sum(sale_price) as sales
from orders
group by year(order_date),sub_category
order by sum(sale_price) desc
)
, xyz1 as (
select sc,
sum(case when yr=2023 then sales else 0 end) as sales_2023,
sum(case when yr=2022 then sales else 0 end ) as sales_2022
from xyz 
group by sc
having sum(case when yr=2023 then sales else 0 end ) >
sum(case when yr=2022 then sales else 0 end) 
)
select sc,
sales_2023,
sales_2022,
sales_2023 *100/sales_2022  as percenatge 
from xyz1
group by sc 
order by percenatge desc
















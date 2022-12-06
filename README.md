Technical documentation

Scope:

In this document we are going to show Extraction, transformation and Loading process. we already have a file(.csv) created by extracting the data from data.gov.ie (Ireland’s Open Data Portal). We will then build and execute the transformation of the data, thereby creating database and pushing the target data into it.


Technical Design:

1.	ETL Architecture:
 ![image](https://user-images.githubusercontent.com/105856868/205929248-b923a9b8-0419-4e17-9c24-5855d281f00a.png)


		Fig.: ETL Process Architecture.
a.	Step 1: Extract
The data can be extracted from different sources, in our Assessment we are having (.csv) data exported from Ireland’s Open Data Portal.
 ![image](https://user-images.githubusercontent.com/105856868/205929324-044c6071-eebb-425f-a248-3da190694c88.png)

We have total Rows: 38556 and Columns: 11.





b.	Step 2: Transform
In this process we do Data purification, which tries to pass only "correct" data to the destination, this is a crucial function of transformation because when multiple systems interact, the interfacing and communication between them is a challenge. A character set that is available in one system might not be available in another.
For example in our data we have rows with Null values, invalid characters like: “,” and “-“  and Duplicates rows.

c.	Step 3: Loading
The data is loaded into the eventual target to a data base server, which could be anything from a basic delimited flat file.

Pandas operations detailed for each requirement:
1.	Loading the input file(.csv) into data frames of Pandas
import pandas as pd
df = pd.read_csv("/content/FireBrigadeAndAmbulanceCallOuts.csv")
2.	To show the count of each Null and not-a-Null values in rows respectively
Null rows by columns: df.isnull().sum()
Non Null rows by column: df.notnull().sum()
Null values for all columns: df.isnull().sum().sum()
Group by: df = df.groupby(['Station Area'])['Station Area'].count()
3.	Replacing “,” and "-" (in any column) with an empty string.
df4 = df_2.replace([',','-'],'')
4.	Drop rows of particular columns (AH,MAV,CD) where one row value is NULL
df5 = df4.dropna(subset=['AH', 'MAV', 'CD'])
5.	Drop any duplicate row except the first occurrence
df6 = df5.drop_duplicates()
6.	Calculate time difference between TOC and ORD columns.
a.	Created function for formatting time HH:MM:SS:
def formatTimedelta(td):
    total = td.total_seconds()
    prefix, total = ('-', total*-1) if total < 0 else ('', total)
    h, r = divmod(total, 3600)
    m, s = divmod(r, 60)
    return f"{prefix}{int(h):02d}:{int(m):02d}:{int(s):02d}"
b.	Created temporary columns to store formatted time into them to avoid changes in original columns:
df6['TOC_Temp'] = pd.to_datetime(df6['TOC'])
df6['ORD_Temp'] = pd.to_datetime(df6['ORD'])
c.	calculate time difference using timedeltas function:
df6['Difference'] = (df6['ORD_Temp']-df6['TOC_Temp']).apply(formatTimedelta)
d.	dropping temporary columns from data frames:
newdf = df6.drop(['TOC_Temp', 'ORD_Temp'], axis=1)
7.	Renamed column which has space in its name so that we can push it to the Database table:
newdf.columns = newdf.columns.str.replace('Station Area', 'Station_Area')
8.	Replaced NULL values as 'NA'.
newdf.replace({np.inf: np.nan, -np.inf: np.nan}, inplace=True)
df7 = newdf.fillna(value="NA")



Testing:
1.	Tested Connection between Colab notebook and SQL server
2.	Validated Target table in SQL server.
3.	Tested Virtual machine and Azure data studio connection.

Reflection on learning:
1.	Understanding the requirement of Assessment
2.	Learned new concepts and libraries of pandas to handle the data frames of any flat file.
3.	Create Virtual Machine instance in MS Azure and connect security port 1433(SQL server).
4.	Installing SQL server in Colab and create connection to it with help of commands.
5.	Pushing final data frames into target table of Database with the help of for loop.
6.	Learned how Azure Data studio works.


Azure Virtual machine instance:
 ![image](https://user-images.githubusercontent.com/105856868/205929460-817ffa54-f555-4ce7-8d0b-a1d1cbe1afed.png)



Security port in Azure portal:
 ![image](https://user-images.githubusercontent.com/105856868/205929492-07074d25-80c6-48b8-834c-285bd36ce747.png)









Azure Data studio:
 
![image](https://user-images.githubusercontent.com/105856868/205929540-b95511b9-64fa-49f9-9e48-dcc621811ed3.png)


Reference:
1.	https://en.wikipedia.org/wiki/Extract,_transform,_load#Transform
2.	https://data.gov.ie/dataset?q=callOuts&res_format=CSV&theme=Health&sort=score+desc%2C+metadata_created+desc





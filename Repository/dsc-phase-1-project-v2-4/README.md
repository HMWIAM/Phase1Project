## Moringa School
## Phase 1 Project
In this READ.md file, we shall be breaking down how I handled this project
The full project can be found here
https://github.com/HMWIAM/Phase1Project/blob/main/Repository/dsc-phase-1-project-v2-4/student.ipynb

# Introduction
The task at hand is to educate and recommend to Microsoft executives on the status of the movie industry.
They want to start their own studio, similar to other large companies like Disney or Netflix
However, before any action can be taken, they require a breakdown of the industry.
They also require a good understanding of the kinds of films that this studio should be making, based potential revenue it can generate.

#### Files to be analysed
There are 6 files that are provided and will be using in this project. These are the:
1. bom.movie_gross.csv
2. im.db
3. rt.movie_info.tsv
4. rt.reviews.tsv
5. tmbd.movies.csv
6. tn.movie_budgets.csv


#### Import libraries to be used
```python
import pandas as pd
import numpy as np
import requests
import sqlite3
%matplotlib inline
import seaborn as sns
import matplotlib.pyplot as plt
import matplotlib as mpl
import random
```

## DATA CLEANING
To start, I need to clean the data provided in such a manner that proper analysis and manipulation can be done.
#### Function that will be in use
I have created function that will make data cleaning really fast.  
This function is designed to fill null values in a column with the mode of that column.  
```python
# This function is designed to fill null values in a column with the mode of that column
def fun_mode_fill_null(df, column_name):
    # First calculate the mode value of a string
    mode_value = df[column_name].astype(str).mode()[0]
    # Then fill the null values with the mode value
    df[column_name].fillna(mode_value, inplace=True)
    return df
```
This function is designed to fill null values in a column with the median of that column
```python
# This function is designed to fill null values in a column with the median of that column
def fun_median_fill_null(df, column_name):
    # First calculate the median value of a float
    median_value = df[column_name].astype(float).median()
    # Then fill the null values with the median value
    df[column_name].fillna(value = median_value, inplace=True)
    return df
```
This function is designed to fill null values in a column with the median of that column, with the added benefit of removing figures that block proper execution
```python
# This function is designed to fill null values in a column with the median of that column, with the added benefit of removing figures that block propoer execution
def fun_median_fill_null_prob(df, column_name, problem):
    # First calculate, remove the troublsome figure, then convert to float, then calculate the median
    median_value = df[column_name].str.replace(problem, '').astype(float).median()
    # Then fill the null values with the median value
    df[column_name].fillna(median_value, inplace=True)
    return df
```
This function is designed to detect the number of duplicate values in a column
```python
# This function is designed to detect the number of duplicate values in a column
def fun_duplicate_count(df, columnname):
    # First check if there are any duplicated values in a column
    dupli_count = [df.duplicated(subset = columnname)]
    # Then print how many are present
    print(len(dupli_count))
```
This function is designed to drop duplicate values in a column
```python
# This function is designed to drop duplicate values in a column
def fun_duplicates_drop(df, column_name):
    # Use the drop_duplicates method in a column, then retain the 1st duplicate in that column, then ensure the drop is permanent
    df.drop_duplicates(subset=[column_name], keep='first', inplace=True)
    return df
```
This function is designed to drop entire columns
```python
# This function is designed to drop entire columns
def fun_column_drop(df, column_name):
    # Using the drop method to specify which column to drop in a dataframe
    df = df.drop(column_name, axis=1, inplace = True)
    return df
```
This function is designed to replace values in a column with other values from a column containing similar values
```python
# This function is designed to replace values in a column with other values from a column containing similar values
def fun_replace_colvalues(df, columnnull, column_2):
    # Specify the column with null values,  the input the column with values you want to replace, then fill in the null values with values in the 2nd column
    df[columnnull] = df[columnnull].fillna(df[column_2].fillna(method = 'ffill'))
```
This function is designed to convert dates in a column into days and months
```python
# This function is designed to convert dates in a column into days and months
def fun_date_convert(df, column_name, dayname, monthname):
    # Specify the column containing the dates and use pandas to_datetime method
    df[column_name] = pd.to_datetime(df[column_name])
    # Create new column that will contain the days and months
    df[dayname] = df[column_name].dt.day_name()
    df[monthname] = df[column_name].dt.month_name()
    return df
```
We shall now load and clean the data provided
### Cleaning the bom.movie_gross.csv
We'll start by loading the data.  
```python
# Import data from bom.movie_gross in the zippedData folder
bom = pd.read_csv('zippedData/bom.movie_gross.csv')
bom
```
We'll then check for duplicates and null values and fill them.  
This will be done as below
```python
# Check the missing values for the columns
bom.isna().sum()

# Now we shall check if there are any duplicate values in title as it is the unique column and drop any duplicates
fun_duplicate_count(bom, 'title')
fun_duplicates_drop(bom, 'title')

# Fill the null values in the domestic and foreign gross with the median values
fun_median_fill_null(bom, 'domestic_gross')
fun_median_fill_null(bom, 'foreign_gross')
fun_mode_fill_null(bom, 'studio')

# Create a new column worldwide gross by adding the domestic and foreign gross and filling the  ull values with the median
bom['worldwide_gross'] = bom['domestic_gross'] + bom['foreign_gross']
fun_median_fill_null(bom, 'foreign_gross')

# Check if there is any remaining missing values in the bom
bom.isna().sum()
```
We should no longer have any duplicates and missing values in BOM

### Cleaning the tmdb.movies.csv
We'll start by loading the data. 
```python
# Open and read the file
tmdb = pd.read_csv('zippedData/tmdb.movies.csv')
tmdb
```
We'll then check for duplicates and missing data, filling them if there are any.  
```python
# Check for null values
tmdb.isna().sum()
# There are no null values so we shall not be filling null values

# We shall then drop duplicates using the unique id column as there should only be 1 unique id
fun_duplicate_count(tmdb, 'id')
fun_duplicates_drop(tmdb, 'id')

# We shall now drop columns that are not comprehensible
fun_column_drop(tmdb, 'genre_ids')
fun_column_drop(tmdb, 'id')
fun_column_drop(tmdb, 'Unnamed: 0')

# We will now create 2 new column called Release day and Release month that will be used later in analysis stage
fun_date_convert(tmdb, 'release_date', 'Release Day', 'Release Month')
```
We have now cleaned tmdb.  

### Cleaning the im.db
We'll start by loading the data
```python
# Opening the file
conn = sqlite3.connect('zippedData/im.db')
```
There are multiple tables in this documents so we shall open and clean then.  
#### Cleaning mov_rat
```python
# Inspecting the movie ratings tables
mov_rat = pd.read_sql("""
SELECT *
  FROM movie_ratings;
""", conn)
mov_rat

# Checking if there are missing vales
mov_rat.isna().sum()
#There are no missing values

# We shall then drop duplicates using the unique movie_id column as there should only be 1 unique id
fun_duplicate_count(mov_rat, 'movie_id')
fun_duplicates_drop(mov_rat, 'movie_id')
```
#### Cleaning mov_rat
```python
# Inspecting the movie basics tables
mov_bas = pd.read_sql("""
SELECT *
  FROM movie_basics;
""", conn)
mov_bas

# Checking if there are missing vales
mov_bas.isna().sum()

# We shall start with replacing missing values in original title with those from primary title
fun_replace_colvalues(mov_bas, 'original_title', 'primary_title')

# We shall the replace missing vales from runtime with their median and genres with the mode
fun_median_fill_null(mov_bas, 'runtime_minutes')
fun_mode_fill_null(mov_bas, 'genres')

# We shall check and remove duplicate using the movie_id unique identifier
fun_duplicate_count(mov_bas, 'movie_id')
fun_duplicates_drop(mov_bas, 'movie_id')

# Check if there are any missing values left
mov_bas.isna().sum()
```
#### Cleaning mov_akas
```python
# Opening and inspecting the movie akas
mov_akas = pd.read_sql("""
SELECT *
  FROM movie_akas;
""", conn)
mov_akas

# Checking for missing values
mov_akas.isna().sum()

#There are alot of missing languages  at 87% so we will drop the column
fun_column_drop(mov_akas, 'language')
#There are alot of missing types  at 50% so we shall drop this column too
fun_column_drop(mov_akas, 'types')
#There are alot of missing attributes  at 95% so we shall drop this column too
fun_column_drop(mov_akas, 'attributes')
mov_akas.isna().sum()

# We will fill the missing regions with the mode
fun_mode_fill_null(mov_akas, 'region')
fun_replace_colvalues(mov_akas, 'is_original_title', 'title')
mov_akas.isna().sum()
```
#### Cleaning mov_princi
```python
# Opening and inspecting movie principals
mov_princi = pd.read_sql("""
SELECT *
  FROM principals;
""", conn)
mov_princi

# Checking for missing values
mov_princi.isna().sum()

fun_column_drop(mov_princi, 'job') #There are 82% missing values
fun_column_drop(mov_princi, 'characters') #There are 62% missing values
mov_princi.isna().sum()

# We shall not be removing duplicates as there seems people can appear more than once for the same movie
```

#### Cleaning mov_pers
```python
# Opening and inspecting movie persons
mov_pers = pd.read_sql("""
SELECT *
  FROM persons;
""", conn)
mov_pers

# Checking for missing values
mov_pers.isna().sum()

# The birth and death column have many missing values so we shall drop them
fun_column_drop(mov_pers, 'birth_year')
fun_column_drop(mov_pers, 'death_year')

# We shall fill the primary profession column with the mode
fun_mode_fill_null(mov_pers, 'primary_profession')
mov_pers.isna().sum()

# There should only be 1 person id in this table so we shall remove duplicates
fun_duplicate_count(mov_pers, 'person_id')
fun_duplicates_drop(mov_pers, 'person_id')
```
#### Cleaning the rt.movie_info.tsv
```python
# Opening and inspecting rt movies information
rt_mov = pd.read_csv('zippedData/rt.movie_info.tsv', sep='\t')
rt_mov

# Checking for missing values
rt_mov.isna().sum()

# There are 68% missing values in studio so we shall drop it
fun_column_drop(rt_mov, 'studio')

# Filling in null in currency with mode
fun_mode_fill_null(rt_mov, 'currency')

# We shall create new column to see the days and months of release for movies
fun_date_convert(rt_mov, 'theater_date', 'Theater Day', 'Theater Month')
fun_date_convert(rt_mov, 'dvd_date', 'DVD Day', 'DVD Month')

#Making runtime and box office cleanable
rt_mov['runtime'] = rt_mov['runtime'].str.replace(' minutes', '').astype(float)
rt_mov['box_office'] = rt_mov['box_office'].str.replace(',', '').astype(float)

# nan appear to be registered as an director name. We shall replace it showing as a missing value
rt_mov['director'] = rt_mov['director'].replace('nan', np.nan)
rt_mov['writer'] = rt_mov['writer'].replace(np.nan)

#Filling in all the null values
fun_mode_fill_null(rt_mov, 'rating')
fun_mode_fill_null(rt_mov, 'genre')
fun_mode_fill_null(rt_mov, 'director')
fun_mode_fill_null(rt_mov, 'writer')
fun_median_fill_null(rt_mov, 'runtime')
fun_median_fill_null(rt_mov, 'box_office')
fun_mode_fill_null(rt_mov, 'theater_date')
fun_mode_fill_null(rt_mov, 'dvd_date')
fun_mode_fill_null(rt_mov, 'Theater Day')
fun_mode_fill_null(rt_mov, 'Theater Month')
fun_mode_fill_null(rt_mov, 'DVD Day')
fun_mode_fill_null(rt_mov, 'DVD Month')
rt_mov.isna().sum()
# We shall leave the null values in synopsis as it's not possible to replicate unique sentences
```
##### Cleaning the rt.reviews.tsv
```python
# Opening and inspecting rt reviews
rt_rev = pd.read_csv('zippedData/rt.reviews.tsv', encoding = 'latin-1', sep='\t')
rt_rev

# Checking for missing values
rt_rev.isna().sum()

# Adding day and month columns
fun_date_convert(rt_rev, 'date', 'Day', 'Month')

#This column is filled with both letter and number making it difficult to analyse so we shall drop it and use fresh columns as an alternative
fun_column_drop(rt_rev, 'rating')

# We shall fill the null values of publishers and critics with the mode
fun_mode_fill_null(rt_rev, 'publisher')
rt_rev.isna().sum()
# We shall not replace the reviews or critics as they have as they are unique to each other

# We shall not remove duplicates as critics appear more than once reviewing different movies
```
#### Cleaning the tn.movie_budgets.csv
```python
# Opening and inspecting tn.movie_budgets.csv
tn = pd.read_csv('zippedData/tn.movie_budgets.csv', encoding = 'latin-1')
tn

# Checking for null values
tn.isna().sum()

# Removing duplicates using unique identifier id
fun_duplicate_count(tn, 'id')
fun_duplicates_drop(tn, 'id')

# Creating a new column for Release days and months
fun_date_convert(tn, 'release_date', 'Release Day', 'Release Month')
# Checking for null values
tn.isna().sum()
# Creating a new column foreign gross assuming worldwide gross contains both domestic andforeign gross
tn['production_budget'] = [float(x.replace('$', '').replace(',', '')) for x in tn['production_budget'] ]
tn['domestic_gross'] = [float(x.replace('$', '').replace(',', '')) for x in tn['domestic_gross'] ]
tn['worldwide_gross'] = [float(x.replace('$', '').replace(',', '')) for x in tn['worldwide_gross'] ]
tn['foreign_gross'] = tn['worldwide_gross'] - tn['domestic_gross']
```
We are now done with Data Cleaning

# EXPLORATORY DATA ANALYSIS
In this section, we shall analyse the cleaned data and visualize them in a manner than is understandable.

## Finance Analysis
Under the Finance section, we shall determine what the expected and mean ROI from the doestic gross, foreign gross, worldwide gross mainly using the TN and BOM table.  


We will start by loading the data
```python
bom
tn
```
Then, we will determine the ROI from the TN table as follows
```python
tn['ROI'] = (tn['worldwide_gross'] - tn['production_budget']) / tn['production_budget']
tn
```
We now have the ROI, next we will find the mean of foreign gross, domestic gross, worldwide gross and production budget.
```python
#We shall the use the mean method to get mean using the tn table
movie_ROI = tn['ROI'].mean()
budget_mean = tn['production_budget'].mean()
foreign_mean = tn['foreign_gross'].mean()
domestic_mean = tn['domestic_gross'].mean()
worldwide_mean = tn['worldwide_gross'].mean()
```
Using this, we now visualize the data.
```python
# I will start visualization by making all relevant information a list
production_budget_list = list(tn['production_budget'])
ROI_list = list(tn['ROI'])
foreign_gross_list = list(tn['worldwide_gross'])
domestic_gross_list = list(tn['domestic_gross'])
worldwide_gross_list = list(tn['worldwide_gross'])

# I will specify the size of figure using figsize
fig = plt.figure(figsize=(10, 8))

# I'll create scatter plot with production budget on the x-axis, worldwide gross on the y-axis, and size of the dots proportional to ROI
plt.scatter(production_budget_list, worldwide_gross_list, s = np.abs(ROI_list)*100, c = np.sign(ROI_list), alpha=0.5, cmap='coolwarm')
plt.title('Relationship between ROI, Gross, and Budget', fontsize=20)
plt.xlabel('Production Budget', fontsize=16)
plt.ylabel('Worldwide Gross', fontsize=16)

# I'll the add the mean values to the plot
plt.axvline(x=worldwide_mean, color='y', linestyle='--', label = 'Worldwide Gross Mean')
plt.axhline(y=budget_mean, color='g', linestyle='--', label = 'Budget Mean')
plt.axvline(x=movie_ROI, color='r', linestyle='--', label='ROI Mean')
plt.scatter(worldwide_mean, budget_mean, color='r')

# add colorbar to show the mapping between the color and ROI values
cbar = plt.colorbar()
cbar.ax.set_ylabel('ROI')

plt.legend()
plt.show()
```
## How well do genres perfrom at the box office?
We can answer this by plotting a bar graph as follows.  
```python
# I'll create a bar graph to show the success of a rating
fig = plt.figure(figsize=(10, 6))

# calculate the mean gross for each genre
mean_gross_by_genre = rt_mov.groupby('genre')['box_office'].mean()

# get the top 10 genres by mean box office revenue
top_10_genres = mean_gross_by_genre.nlargest(10)

# create a bar plot for the top 10 genres
top_10_genres.plot(kind='bar')
plt.xlabel('Genre')
plt.ylabel('Box office revenue')
plt.title('Top 10 genres by mean box office revenue', fontsize=20)
plt.xticks(rotation=90)
plt.show()
```
This shoud make a bar graph showing the relationship between genres and box office revenue.  


Know we can see that, from the plot the relationship between the gross, production budget and ROI.  
## Movie characterist analysis
In order for a studio to generate the highest ROI possible, it is best to look into the characteristics of the most popular movies.  
### What is the most popular genre?
To answer this, we will analyse how many films were made on a particular film.  
``` python
mov_bas['genres'].value_counts()[0:10].plot(kind='bar')
plt.xlabel('Genres')
plt.ylabel('Number of Movies')
plt.title('Number of Movies by Genres', fontsize=20)
plt.show()
```
This piece of code will show us what is the genre that has the most number of films.  
We would also like to know the runtime based on genre.  
```python
# Group the DataFrame by genre and calculate the mean runtime for each genre
mean_runtime_by_genre = mov_bas.groupby('genres')['runtime_minutes'].mean().reset_index()

# Sort the resulting DataFrame in descending order by mean runtime and select the top 10 genres and their mean runtimes
top_genres = mean_runtime_by_genre.sort_values(by='runtime_minutes', ascending=False).head(10)

# Create a bar chart using Seaborn
sns.set(style="whitegrid")
ax = sns.barplot(x="genres", y="runtime_minutes", data=top_genres)

# Set chart titles and labels
ax.set_title("Top 10 Genres by Average Runtime", fontsize=20)
ax.set_xlabel("Genre")
ax.set_ylabel("Average Runtime (minutes)")
plt.xticks(rotation=80)
```
This would produce a bar chart that shows us the runtime of different genres.  
### What is the most lucrative rating ?
We would also like to know which ratings earns the most in the box office.  
```python
Rating_list = list(rt_mov['rating'])
Box_office_list = list(rt_mov['box_office'])
# I'll create a bar graph to show the success of a rating
fig = plt.figure(figsize=(10, 6))

# Create a bar graph
plt.bar(Rating_list, Box_office_list)

# Add labels and titles
plt.xlabel('Ratings')
plt.ylabel('Box office')
plt.title('How well do ratings do in the box office', fontsize=20)

# Display the graph
plt.show()
```
We now have a bar chart showing us how much each genre gained from the box office.  
### Do critic ratings influence earnings?
To answer this question, we will need to merge rt_rev and rt_mov as follows.  
```python
# I'll try merging the tables rt_rev and rt_mov and see how to count number of rotten and fresh
# Group tableB by ID and Review, count the number of occurrences of each, and unstack the results
Fresh_reviews = rt_rev.groupby(['id', 'fresh'])['fresh'].count().unstack()
# Rename the columns to 'bad' and 'good'
Fresh_reviews.columns = ['Fresh', 'Rotten']
# Merge the counts with tableA on the 'ID' column
merged_table = pd.merge(rt_mov, Fresh_reviews, on = 'id')

rt_mov = merged_table
```
We can then visualize this by creating a scatter plot as follows.  
```python
# create a figure with a specified size
fig = plt.figure(figsize=(10, 8))

# Extract the columns of interest
gross = rt_mov['box_office']
good_reviews = rt_mov['Fresh']
bad_reviews = rt_mov['Rotten']

# Create a scatter plot with regression line
sns.regplot(x=good_reviews, y=gross,marker='o', color='green', label='Good Reviews')
sns.regplot(x=bad_reviews, y=gross, marker='x', color='red',label='Bad Reviews')

# Add axis labels and a title
plt.xlabel('Number of Reviews')
plt.ylabel('Box Office Revenue')
plt.title('Movie Reviews vs. Box Office Revenue', fontsize=20)

# Add a legend
plt.legend()

# Display the plot
plt.show()
```
We should now see a scatter plot and see the relationship between critic reviews and box office revenue.  

## Human Resources
### How many kinds of professional are in the markets?
The movie industry has many kinds of professions so we would like to know how the labour supply looks like.  
We can answer this question as follows.  
```python
mov_princi['category'].value_counts().plot(kind='bar')
plt.xlabel('Professions')
plt.ylabel('Number of Professionals')
plt.title('Number of Professionals', fontsize=20)
plt.show()
```
This bar graph shows us the labor situation according to the data provided.
### How many professionals do we need per movie?
As we would like to create movies, we would further want to see what is the staff requirement per film.  
To do this, we need to merge mov_bas and mov_princi as follows.  
```python
# I want to count the number of professional that have appeared per movie
# Merge the two tables by ID
people_run = pd.merge(mov_bas, mov_princi, on='movie_id')

# group by movie and profession, and count the number of occurrences
counts = people_run.groupby(['primary_title', 'category']).size().reset_index(name='Count')

# pivot the table so that each profession becomes a separate column
pivot_table = counts.pivot_table(values='Count', index='primary_title', columns='category', fill_value=0)

# merge the pivot table with the original table
result = pd.merge(mov_bas, pivot_table, left_on='primary_title', right_index=True)
```
Next we need to plot this merged table.  
```python
# I put the count in a table
# melt the pivot table to make plotting easier
melted_df = pivot_table.reset_index().melt(id_vars=['primary_title'], var_name='profession', value_name='count')

# create a bar plot
sns.barplot(x='profession', y='count', data=melted_df)

# set the chart title and labels
plt.title('Number of Times Each Profession Appears in a Movie')
plt.xlabel('Profession')
plt.ylabel('Count')

# rotate the x-axis labels to prevent overlap
plt.xticks(rotation=90)

# display the plot
plt.show()
```
We can now see how many staff member we need per movie.

### Does director or writer choice affect revenue?
To answer this, we will need to create to bar charts that show how much directors and writer attract at the box office.
```python
# group by director and sum the revenue
revenue_by_director = rt_mov.groupby('director')['box_office'].sum().reset_index()

# sort the data by revenue in descending order and select the top 10 directors
top_directors = revenue_by_director.sort_values('box_office', ascending=False).iloc[1:11]

# create a bar plot
plt.bar(top_directors['director'], top_directors['box_office'])

# add titles and labels
plt.title('Top 10 Directors by Revenue')
plt.xlabel('Director')
plt.ylabel('Total Revenue')
plt.xticks(rotation=90)

# show the plot
plt.show()
```
   
```python
# group by director and sum the revenue
revenue_by_writer = rt_mov.groupby('writer')['box_office'].sum().reset_index()

# sort the data by revenue in descending order and select the top 10 directors
top_writer = revenue_by_writer.sort_values('box_office', ascending=False).iloc[1:11]

# create a bar plot
plt.bar(top_writer['writer'], top_writer['box_office'])

# add titles and labels
plt.title('Top 10 Writers by Revenue')
plt.xlabel('Writers')
plt.ylabel('Total Revenue')
plt.xticks(rotation=90)

# show the plot
plt.show()
```
We should now be able to see the relationship between box office and writer/ directors.





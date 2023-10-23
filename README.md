# Welcome to the Analysis of Shark Attacks around the world!

![YO](https://media.giphy.com/media/xvDqUkDJNRLot0aAOj/giphy.gif)

__[This Analysis will make use of the dataset provided in Kaggle via this link](https://www.kaggle.com/datasets/teajay/global-shark-attacks)__

## This README file
In this Readme file you will get a gist of how I approached the analysis step by step. 
I'll try to put in as simple terms as possible and whenever an explanation requires code or is aimed at people with a basic knowledge of python, you will find the symbol "🤖" behind it so you know you can skip those sections if you don't find it relevant reading/it's too technical. A brief general explanation of the process can always be found regardless of its technicality. The emoji 🐤 denotes that the lingo used is generic enough again to be consumed by people without a technical knowledge in Python. 

![study](https://media.giphy.com/media/1jaJf7liv7Dl5SrrTU/giphy.gif)

## 1.  The proposed hypothesis
The following are the hypothesis I commit to analyse, accomplishing this will require cleaning the original dataset 🤖 and making use of some powerful tools from Python like Panda Libraries, Seaborn and Matplotlib:

1 How old are the biggest victims of shark attacks in Australia?
2 What time of the day are sharks the angriest?
3 What countries like to play with fire the most?
4 When were women the most targeted in USA between 1995-2018?

As a sum up, the variables that I will work on are:
##### Country, Age, Fatal(Y/N), Time, Year, Sex, Type

![cleaning](https://media.giphy.com/media/hGxFZVEZbWfRcCbWyZ/giphy.gif)
Yup, some people clean data, others pollute the oceans :')

## 2. The data cleaning process
Before starting to correlate the variables and plotting actual charts, these variables require some data cleaning. This comes as the original dataset is not 100% accurately maintained/data-introduced. A practical example would be:
Country - you would expect this column to entail exclusively non-repeated names of existing countries, but in reality, we can find entries like:
    INDIAN OCEAN? (not a country)
    UNITED ARAB EMIRATES (UAE) (and later on UNITED ARAB EMIRATES, ie, they are not uniform)
    EQUATORIAL GUINEA / CAMEROON (two countries, not a single one)

Cleaning this data is a time-consuming but rewarding process as having correct entries will bring us closer to the "real" answers.
Let's start by delving into the "Country" column data cleaning process. 

Important assumptions:
- I have actually created more than 1 cleaned datasets, why?
    In Q1 (question 1) I will be using the information in columns Country and Age. In Q2 I am using Fatal and Time columns. In case there are rows in Time with missing information where I can't work that missing piece around, I will discard (drop) it, but it might be that the information for Country and Age (useful for Q1) are there. I would be missing valid observations for one question in favor of another!
- Some additional considerations will be explained on how I approached the data management in this doc.

### Cleaning "Country"
In order to achieve so, I started by importing the original attacks.csv database. This is the file that will be object of scrutiny.
I've listed all the unique entries in this column to better assess how to tackle this data. From here I noticed what values I should be working on individually and transform them into "correct" values (by standardizing the way they were spelt, checking for more information in other columns whenever the Country one was not clear and correcting introduction typos). 

🤖 (technical explanation of the approach)
Done so printing the unique array from column "Country".

After this I've created a dictionary (countries_correct_names_dict) with the "wrong" terms to be found in the uniques Array and their corresponding correct values, example: 'NEVIS':'SAINT KITTS AND NEVIS' (WRONG:RIGHT). 

Information from other columns (Location or Area for example) was also used to better fill the wrong information in Country. I'm making usage of the loc method to replace it in the several instances I found required my attention:
sharks_q1.loc[sharks_q1['Location'] == '360 miles north of Ascension Island', 'Country'] = 'ASCENSION ISLAND' where I am locating the labelled String "360 miles north of Ascension Island" within the Location column and in the very same row, replacing the existing value in the "Country" column with "ASCENSION ISLAND". 

🐤 As I am following the process I went through during my analysis, I should report that - after checking for what kind of data I am working with (🤖 making use of methods like value_counts, duplicated(), describe, etc.) my first step should have been to delete information that's not really information. I mean: the original data set goes down to row 25724 but in reality the rows stopped being filled with actual data since row 8703, more than this(!) from row 6303 up to 8703 there's not data, just zeros. I've deleted this irrelevant amount of (mis)information - 🤖 by using a simple drop and the index of what I want to keep off my working data: sharks_q1.drop(sharks_q1.index[6244:], inplace=True).

🐤 Still some rows might contain duplicated information. For the sake of simplicity, I assumed that 2 rows are duplicates whenever the info in the columns 'Case Number', 'Date' and 'Country is the same between them. I removed this information and reset the index of the now worked dataset (this means that - if I had 5 rows, respectively indexed from 1 to 5 and I deleted a duplicate in row number 3, my last row would still be the index 5, but I would be missing an index 3. By resetting the index, I'll make sure that I am "pullin" the subsequent rows to the deleted one so that the former index 4 is now the (deleted) 3 and the data goes up till index 5.
🤖 I'm deleting the duplicates and resetting the index by using the drop method, I'm keeping the first row of data, whenver there's a duplicate, example:
sharks_q1_countries_cleaned = sharks_q1_countries_cleaned.reset_index(drop=True)
sharks_q1_countries_cleaned = sharks_q1_countries_cleaned.drop_duplicates(subset=['Case Number', 'Date', 'Country'], keep='first')


### Cleaning "Age"
Age requires cleaning as entries like:
    Elderly
    ? & 19
    25 to 35
    MAKE LINE GREEN
are not actual ages. Also, requires an decision on the missing "Nan" values.

🤖 Using the sum of the isna (remembering isna converts the series into Booleans assuming True/False (1/0) and then sums them) I see I have 2734 missing values in this column. 
sharks_q1["Age"].isna().sum()

I'm plotting a sample of the dataframe sharks_q1[sharks_q1['Age'].isna()] to check how to generally approach these values 🐤 - that is saying, I need to have an overview on the dataset where Age is a missing value and see if I can get some additional information in other columns to assess my next step (it could be for example that the person introducing the data wrote the Age in another column for example). 

🐤 Just so I can later one using the Age in my plots as a quantitative variable (numerical one), I first want to make sure this is a number and that "82" is not a text (string). I will convert every possible information into numbers and be left out with "textual" information that I need to tackle in another way. 

🤖 I will be trying to convert every entry into an integer, in case that's not possible (Age = "eighteen" instead of 18) I'll just return whatever is there in the column and later on deal with it (stay tunned, I'll explain). Finally applying this to the dataframe sharks_q1 using the apply method. 
def converting_the_ages_to_integer(age):
    try:
        return int(age) 
    except:
        return age
sharks_q1['Age'] = sharks_q1['Age'].apply(converting_the_ages_to_integer)

🐤 I will be left with some information that is not comprised of numbers. Important consideration:
- I dropped information that is not exact, example: Age = "Teen", "12 or 13", "30s". In my analysis I want to have the age for every person involved in sharks incidents as accurately as possible. 

Final step in age: I want to make sure that I don't have "suspicious" values in the age, meaning - negative ages or Age =146 (in case someone mistyped a "1" in excess for example). I'm setting a range of "acceptable" ages where people can be between babies (>0 years old) or elderly people to a maximum of 100yo. 


🤖 I've defined a condition, checked the shape of the new dataframe and as it had the same shape with or without the condition I ended up not needing to remove "strange" ages:
condition_age_is_reasonable = (sharks_q1['Age'] >= 0) & (sharks_q1['Age'] <= 100)
valid_age_rows = sharks_q1[condition_age_is_reasonable]

Once again, I removed duplicates and reset the index. 

🐤 IMPORTANT note: as mentioned in the beginning - I will be exporting/working with several cleaned datasets (dataframes). Anytime I get to the end of the cleaning of a variable, I save the current data set:

🤖 sharks_q1_countries_and_age_cleaned = sharks_q1.copy() (right after this I "freeze" the dataframe by commenting it "#"
    

### Cleaning "Fatal (Y/N)"
My starting point will be the dataset where I cleaned the excessive rows at the end of it (as mentioned earlier). I will be working over a new dataset sharks_q2.
Since this variable is used in Q2 (question 2) but shares no other columns with Q1, there are no economies of scale I can take from previous work.

This column does not contain exclusively "Y" and "N" values, but rather:
['N' 'Y' nan 'M' 'UNKNOWN' '2017' ' N' 'N ' 'y'] - all of the between '' are existing entries in the column.

##### Dealing with " N"
🐤This is just a "N" in disguise because it has an extra spacing in the beginning. I've removed it and turned it into "N":

🤖sharks_q2['Fatal (Y/N)'] = sharks_q2['Fatal (Y/N)'].str.strip()

##### Dealing with "y"
🐤 I've made everything upper case so the "y" turns into a "Y".
🤖 Using an upper method: sharks_q2['Fatal (Y/N)'] = sharks_q2['Fatal (Y/N)'].str.upper()

##### Dealing with "2017"
🐤 Went to check for further information in the other columns / PDF and got to the conclusion it was not a fatal incident, so I changed this entry:
🤖 Using an the Loc method to locate the "2017" value in this column and replacing it with an "N" sharks_q2.loc[sharks_q2["Fatal (Y/N)"] == "2017", "Fatal (Y/N)"] = "N"

##### Dealing with "UNKNOWN"
🐤 This requires understanding what information the rest of the columns and/or supporting PDFs give us. The "injuries" column was very helpful in giving us some extra info. I've printed the individual results in this column to understand what to do with them individually. I realized the best approach was to use Regular Expressions to tackle this. This is - I went to check for some "buzzwords" that would suggest there was indeed a fatality: death|drowned|died|post mortem|drowning|fatal
At the same time, I am telling Python "if there is the world 'fatal' in the injury, consider that was a "Y" (fatality) but if there is "non-fatal" or "non fatal" this is NOT a fatality. 

🤖 I am using regex to tackle these buzzwords:
fatal_rows = sharks_q2['Injury'].str.contains(
    fatal_keywords, case=False, na=False, regex=True
) & ~sharks_q2['Injury'].str.contains(
    r"\b(?:non-fatal|not[- ]?fatal)\b", case=False, na=False, regex=True
)

\#Update "Fatal (Y/N)" for rows that meet the criteria to "Y"
sharks_q2.loc[fatal_rows, 'Fatal (Y/N)'] = 'Y'

🐤 After this much scrutiny I assume that the remaining values are missing information and I can not guess if they were indeed casualties. I dropped (deleted) them. 


### Cleaning "Time"
🐤 The time was mainly introduced in the form "HHhMM" (HOUR h-separator MINUTE). I am categorizing the Time into 4 different clusters and into a new column called "Time of Day":

Morning     5 am to 12 pm (noon)
Afternoon     12 pm to 5 pm
Evening     5 pm to 9 pm
Night         9 pm to 4 am
(source: www.britannica.com)

Once again I am using Regular Expressions to check what cluster the first group of information "HOUR" falls within (example: 11h32 would fall withing "Morning" as 11<12 (12 being the limit I set for the Morning cluster).


🤖  match = re.match(r'(\d{1,2})h(\d{2})', time_str)    
    if match:
        hour = int(match.group(1))
        minute = int(match.group(2))
        # Convert to a single hour value (0-23)
        hour_value = hour + minute / 60

        # Categorize based on time ranges
        if 5 <= hour_value < 12:
            return "Morning"
        elif 12 <= hour_value < 17:
            return "Afternoon"
        elif 17 <= hour_value < 21:
            return "Evening"
        else:
            return "Night"
    else:
        return "Unknown"
        
If the return is "unknow" I will deal with that information in another way.

🐤 The Regex method does not accomplish cleaning all of the information as not everything was introduced in HHhMM. I am left out with a large amount of data data is not time in this format. Before I scrutinize it in more detail, I am looking for the keywords in this column:

For a matter of simplicity:
If the entry contains the text "morning" - I assumed the cluster should be morning
If the entry contains the text "afternoon" - I assumed the cluster should be afternoon
(...) evening
(...) night


🤖Allocating these info:
time_contains_word_afternoon = sharks_q2['Time'].str.contains('afternoon', case=False, na=False)
sharks_q2.loc[time_contains_word_afternoon, 'Time of Day'] = "Afternoon"

🐤I am still left out with information that's not categorized:
'Shortly before 12h00', 'After noon', '1300','Sometime between 06h00 & 08hoo', '0830', 'Just before noon', (...)

just like I've done previously I am tackling these individually. 
🤖 Created a dictionary to make this match and and then replaced the information. 


### Cleaning "Type"
IMPORTANT CONSIDERATION:
I am creating the categories 'Boating', 'Unprovoked', 'Provoked', 'Questionable'.
The first approach was to convert some values into "Boating" (Boat', 'Boatomg').

Sea Disasters showed up often as a type but for my analysis I will also be consider these as "Unprovoked". As a matter of fact, I assumed that activitites that do pose a risk to the person's integrity are considered provoked (ex: Spearfishing / Scuba diving, Shark diving, Cage diving). 

🤖 I have achieved the data homogeinity but applyin the same process with the dictionaries as before. Dropped values whenever - even after scrutinizing the "Activity" column I could not get clear information that allowed me to categorize type. 


### Cleaning "Year"
My initial approach to year was to print every unique year and get a grasp on whether the entries make sense. I speculated that years like 5 or 500 would be typos but in reality and after checking these few examples, one can see they did indeed occur. Year 0 on the other hand is repeated several times as a typo. 

As I check for additional information in the "Date" column, I notice that these 0s are indeed not accurate at all. I removed them from my analysis, some examples:
Ca. 336.B.C..', '493 B.C.', 'Ca. 725 B.C.', 'Before 1939','1990 or 1991', 'Before 2016', 'Before Oct-2009', 'Before 1934',...


### Cleaning "Sex"
The last variable to be cleaned, sex should be comprised of either "M", "F" or "No info" values, in fact before cleaning these are the existing entries:
['F', 'M', nan, 'M ', 'lli', 'N', '.']

##### Dealing with "M " 
An extra spacing was to be found in this entry which let me to homogenize the information
##### Dealing with "N"
No information could be found in other columns or PDFs, I've categorized this entry as "No info"
##### Dealing with "lli"
The PDF suggested the victim was a man, so I changed this information accordingly to "M"
##### Dealing with "lli"
This row is respectful to several victims with no further detail on the information present. I've categorized this entry as "No info"
##### Dealing with "Nan"
To tackle these entries I have once again employed the Dictionary + Replace approach, focusing on the categorical information that suggested the victim(s) was a Woman as that's what I will be using in my analysis.

![them deets](https://media.giphy.com/media/6bCqBl45PAQSqWmShs/giphy.gif)


## 3.  The conclusions

### 1 How old are the biggest victims of shark attacks in Australia?

In the first question I propose to analyse, I seek to get a perspective on the Age distribution of shark attacks in particular in Australia.
This is a country very well known for its deadly wildlife while at the same being one of the biggest coastal countries in the world, water activities are abundant.

As one would probably expect, the youger people are the most active in water activities and with so, also the riskier to be involved in shark incidents. The conclusion points towards a positively skewed distribution of shark attacks where the younger are also the more often actors in such a sea-drama. People close to 20 years old are by afar the most savaged by sharks. 

### 2 What time of the day are sharks the angriest?

The second research topic has to do with the fatality of the attacks, more precisely we are aiming at finding out when, during the day are sharks the deadliest. 

Maybe not completely surprisingly either, sharks are the deadliest during the day as in - during the day is usually also when people are the most active in the water (I'm lacking data to support this thesis). The conclusions are the same for non-deadly attacks.


### 3 What countries like to play with fire the most?

Well, actually, what countries like to play in the water the most? Here we're looking at what countries provoke incidents the most. 
USA and Australia stand out the most with a large amount of observations in absolute terms, I reckon not only because they are very coastal countries but also due to a bigger extent of documentation.

In relative terms I am assessing how much of the total incidents were provoked as a proportion of the total. This is a simple division "Provoked" / "Total incidents"
In this department the top 5 is awarded to:
1. UK
2. Tanzania
3. Malasya
4. Palau
5. China


### 4 When were women the most targeted in USA between 1995-2018?

Finally, we want to assess the distribution of shark attacks in the USA throughout the years where a woman was the victim. 
Fairly honestly I was hoping to find something shocking and then try to find an explanation for that, so my research method went the other way around - I did not have a hypothesis, rather wanted to find something and then justify. I did not.

There seems to be a crescendo in the attacks on women from 2013-2017 but still a very narrow time frame to draw conclusions. There seemed to be a rising and then decreasing wave of attacks that peaked in 2007, then again with no relevant amount of attacks to draw conclusions from.
Overal per year there are not that many reported attacks on women per year in the USA where we see an all time (for this timeline) high in 2017 with 19 attacks.


# Much love for sharks 🦈 ❤️
P.s.: Fossil evidence suggests that sharks have existed on Earth for over 400 million years, which means they've been around longer than dinosaurs and even trees. They're true survivors of evolution
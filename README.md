# Music Listening Habits During Recessions
This project sought to tackle how signals of poor economic conditions affect individual music listening habits across genres. More practically, this was conducted to practice pulling raw API data, cleaning it, merging it with an external source, and turning it into something regressible.

## Overview
This project was for a class Data Analytics (R and Python)(ECO 590) with a large emphasis on developing the technical skills necessary to take a real, messy, API and turn it into a clean dataset which could then be the subject of econometric techniques. Any econometric findings were a reward from conducting sound models and applying them to my data, they were not the point of evaluation. For this project I used the Last.fm API; I pulled weekly music listening history, classified each artist within said listening history into a macro-genre (referred to henceforth as simply, *genre*) using tag-based keyword matching, computed the weekly play share for each genre, merged this data with macroeconomic indicators (recessions, unemployment, inflation, consumer sentiment), and ran linear regressions in R to see how listening patterns change during economic downturns.
**An important note regarding the scope of this project:** The listening data in this project reflects one individual Last.fm user, rj, not an aggregate or population-level measure. This is a single case, *not* a reflection of how people's listening habits change overall as a result of macroeconomic phenomena. Although the scope of data limited  it limited my original presentation's global significance, I am currently (as of 9/9/26) working on scaling this project to a US scale (See Future Improvements section for more info)

*Code was debugged with the assistance of AI tools (Claude); all research design, modeling choices, and interpretation of results are my own.*

## Data
### Source:
Weekly listening history, artist level play counts, and artist tags were pulled from Last.fm API via "requests" Python package. [Last.fm API Documentation](https://www.last.fm/api) Recession indicator, unemployment rate, PCE price index, and University of Michigan consumer sentiment were sourced from Federal Reserve Economic Data (FRED®) API using "fredapi" Python package. [FRED API Documentation](https://fred.stlouisfed.org/docs/api/fred/) [fredapi Python Package information](https://pypi.org/project/fredapi/)

### Time Period:
2005-2025, data was bounded by the start of Last.fm's listening history; FRED data was filtered to match this window.

### Key Variables:
#### From Last.fm:
**Play Share -** a genre's share of a given week's total plays, expressed as a decimal.
**Total Plays -** total plays across all artists in a given week, regardless of genre. 
**Genre -** assigned per artist via keyword-matching on that artist's top Last.fm tag (ex. tags containing "rock," "punk," "grunge," or "britpop" were bucketed into "Rock")

#### From FRED:
**REC -** binary recession indicator (1 = recession, 0 = no recession), NBER/FRED (USREC) 
**UNRATE -** unemployment rate, continuous, FRED (UNRATE) 
**PCE -** Personal Consumption Expenditures price index, logged for % change (inflation), FRED (PCEPI) 
**UMICH -** University of Michigan Consumer Sentiment Index, logged for % change, FRED (UMCSENT)

## How to Reproduce
### Requirements:
Python 3.x with requests, pandas, numpy, matplotlib, seaborn, fredapi
R with tidyverse and stargazer
A free Last.fm API key [(get one here, requires account creation)](https://www.last.fm/login?next=/api/account/create) and a free FRED API key [(get one here, requires account creation)](https://fred.stlouisfed.org/docs/api/api_key.html)

### Getting Data:
The notebook pulls all Last.fm data live via API calls, though a full run will take a while due to the rate-limit delay. The notebook currently pulls weekly data for a single hardcoded Last.fm username. To reproduce for a different listener, swap the user= parameter in the API calls (a more efficient way to do this is to use f' strings and create a user variable that can easily be changed). FRED data is pulled live via fredapi and requires no manual downloads.

## Methodology
### Step 1: Collect Weekly Listening Data
Using Last.fm's tag.getWeeklyChartList endpoint, I first pulled the full list of available weekly date ranges. For each week, I called user.getWeeklyArtistChart to get that week's top artists (capped at 250 per week) and appended every artist name to a running master list across all weeks. The loop structure to get the top artists is essentially the outer loop cycles each week and the inner loop collects every artist for a given week, and moves onto the next week once completed (see screenshot 1.1). This master list intentionally contains duplicates at this stage, since deduplication happens in a later step when building the genre lookup.

<img width="982" height="280" alt="image" src="https://github.com/user-attachments/assets/cb6646aa-3528-4213-8ded-39ea5d5b95a8" />

#### Screenshot 1.1 (Loop Structure)

<img width="211" height="289" alt="image" src="https://github.com/user-attachments/assets/e2b7ff39-c2b4-401c-aea3-dfbeab6762bc" />

#### Screenshot 1.2 (Loop Output, *number denotes week number*)


### Step 2: Create Artist Tag Dictionary and Genre Buckets
In step 2 I created a general dictionary that pairs each artist with their raw, unfiltered tag. The structure of this loop first encodes the artist name to avoid any special character errors (Simon & Garfunkel was breaking my code because of the ampersand, encoding removes that issue) Then, the encoded name is compared to the dictionary, if it is already in the dictionary it is passed, if not the internal loop gets the top tag and pairs it with the artist in the dictionary. The counter is there for me to keep track of how many artists I have, and the pause is an AI suggestion to circumvent my API limit rates. 
From this artist tag dictionary I was able to sort each tag into one of nine genres (Pop, Rock, Jazz, Electronic, Indie, Metal, Classical, Hip Hop, Folk) This step works and, even though it could have been done more efficiently, each of the artists were now lumped into their own genre within a dataframe.

<img width="926" height="482" alt="image" src="https://github.com/user-attachments/assets/805257cb-29cb-48f7-8b21-b72e19d738a6" />

#### Screenshot 2.1 (Loop Structure)

<img width="295" height="281" alt="image" src="https://github.com/user-attachments/assets/ef29849f-6a47-42ea-980e-301c8da0ea0b" />

#### Screenshot 2.2 (Loop Output, *number denotes artist count*)

<img width="981" height="499" alt="image" src="https://github.com/user-attachments/assets/8638d09a-58bd-4fdc-b84b-3863f09a17f1" />

#### Screenshot 2.3

### Step 3: Building Weekly Panel
Now that every artist is mapped to a genre, I looped back through each week, summed plays by genre and found each genre's share of said week's total listening, creating a panel of one row per genre per week. I exported this panel to a .csv file so that I had a jumping off point when working within the same week, and for visualizing and regressing my data. 

<img width="984" height="854" alt="image" src="https://github.com/user-attachments/assets/63c37786-6e0a-427d-8020-cb8d275fcfe8" />

#### Screenshot 3.1

<img width="984" height="160" alt="image" src="https://github.com/user-attachments/assets/685f2486-d634-48b6-8466-91bff0a06e1c" />

#### Screenshot 3.2

<img width="448" height="430" alt="image" src="https://github.com/user-attachments/assets/d4d6228a-416c-4def-b9bf-7c39b2ca5e64" />

#### Screenshot 3.3

### Step 4: Merging with FRED Macro Data
Now that each of the genres had their play shares for each week, I needed to merge it with the macro data from FRED. I pulled each of the respective series using the "fredapi" package, converted them to one data frame that was separated by month and year. I then joined them to the previous data frame which contained artists and genres. I then filtered my data so that I could find data between 2005 and 2025, before saving everything to a .csv file. 

<img width="984" height="432" alt="image" src="https://github.com/user-attachments/assets/4004b628-e64e-4c54-8829-6a0a10bc4833" />

#### Screenshot 4.1

<img width="982" height="625" alt="image" src="https://github.com/user-attachments/assets/51b97c36-aa71-4157-a732-ca1990a704dd" />

#### Screenshot 4.2

<img width="867" height="704" alt="image" src="https://github.com/user-attachments/assets/b6ee78a0-cbd8-46b8-8a3c-90c283a3b1d0" />

#### Screenshot 4.3

### Step 5: Visualizing Listening Patterns Over Time
I built five plots in matplotlib/seaborn: a full genre breakdown over time, a cleaner version limited to the top 4 genres, total monthly listening volume, average monthly listening, and a genre-by-year heatmap. NBER recession periods were shaded on each time-series plot for visual reference.

<img width="687" height="493" alt="image" src="https://github.com/user-attachments/assets/956017c4-b398-4608-bdfc-294b0c09eca8" />

#### Screenshot 5.1

<img width="687" height="483" alt="image" src="https://github.com/user-attachments/assets/1a195d4f-eb41-462f-bbf9-6a1dfdaf3af2" />

#### Screenshot 5.2

### Step 6: Regression Analysis in R
In R, I ran two sets of linear models: one regressing total monthly plays (and log of total monthly plays) on the recession dummy, unemployment rate, and logged inflation/consumer sentiment. The second model regressed each genre's logged play share on the same macro variables plus logged total plays, run separately per genre (Rock, Metal, Jazz, Hip Hop, Pop, Classical) rather than as a single pooled model with genre fixed effects, since separate regressions were more interpretable at this stage of the project.

<img width="401" height="556" alt="image" src="https://github.com/user-attachments/assets/7a25f382-3009-4758-ab69-92e85487dfee" />

#### Screenshot 6.1

<img width="750" height="587" alt="image" src="https://github.com/user-attachments/assets/11ef461a-3d0a-48c2-9113-75470d76ed9f" />

#### Screenshot 6.2

## Findings
Read these as descriptive results about **one individual's listening behavior,** not generalizable claims about how people listen to music during recessions. An individual's listening habits are largely affected by preferences and other factors that would be difficult to control for without survey data. The sample size here is one listener over roughly 20 years, which is enough to fit a regression but not enough to support population-level conclusions.
With that in mind, interpreting these results show interesting trends: total monthly plays dropped by nearly 60% during recession periods in this dataset, and inflation had a notably large negative effect on overall listening volume. Within the top 4 genres the only genre whose listening share increased by during recessions with statistical significance was metal, which increased by 72%; the coefficient here is extremely large, which I hypothesize reflects the listener's preferences. Despite being statistically insignificant, hip hop's play share saw the same relationship as metal with a 9% increase during recessions. Rock and jazz saw a smaller share of listening during recessions, with an 8% decrease and a 21% decrease respectively. An important note for interpretation is that the play share regressions' coefficients reflect a percentage change of a variable which is already a percentage. 

For the total plays regressions


## Planned Extensions
i want to add more variables likes:
**Fed policy variables:** and heres why

This project was completed independently for a data analytics coding course (Data Analytics (R and Python)(ECO 590)); the FRED-based econometric analysis was an extension beyond the course's core requirements, which focused primarily on the data collection, cleaning, and the mechanics of regression in R.

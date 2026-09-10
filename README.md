# Music Listening Habits During Recessions
This project sought to tackle how signals of poor economic conditions affect individual music listening habits across genres. More practically, this was conducted to practice pulling raw API data, cleaning it, merging it with an external source, and turning it into something regressible.

## Overview
This project was for a class Data Analytics (R and Python)(ECO 590) with a large emphasis on developing the technical skills necessary to take a real, messy, API and turn it into a clean dataset which could then be the subject of econometric techniques. Any econometric findings were a reward from conducting sound models and applying them to my data, they were not the point of evaluation. For this project I used the Last.fm API; I pulled weekly music listening history, classified each artist within said listening history into a macro-genre (referred to henceforth as simply, *genre*) using tag-based keyword matching, computed the weekly play share for each genre, merged this data with macroeconomic indicators (recessions, unemployment, inflation, consumer sentiment), and ran linear regressions in R to see how listening patterns change during economic downturns.
**An important note regarding the scope of this project:** The listening data in this project reflects the listening habits of one user, rj, who also is the creator and co-founder of Last.fm ; this is *not* an aggregate or population-level measure. This is a single case, *not* a reflection of how people's listening habits change overall as a result of macroeconomic phenomena. Although the scope of data it limited my original presentation's global significance, I am currently (as of 9/9/26) working on scaling this project to a US scale (See Future Improvements section for more info)
**A note regarding the code for this project:** I utilized Jupyter Notebook for all of the Python code. There may be slight errors when utilizing Microsoft vsCode as different sections are run sequentially and allowed to complete before moving on.  

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
The notebook pulls all Last.fm data live via API calls, though a full run will take a while due to the rate-limit delay. The notebook currently pulls weekly data for a single hardcoded Last.fm username. To reproduce for a different listener, swap the user= parameter in the API calls (a more efficient way to do this is to use f' strings and create a user variable that can easily be changed). FRED data is pulled live via fredapi and requires no manual downloads. I reference the .csv file I created when constructing the graphs. In order to avoid errors, replace my path name with your own file path.

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

#### Screenshot 2.3 (Genre Bucket Creator)

### Step 3: Building Weekly Panel
Now that every artist is mapped to a genre, I looped back through each week, summed plays by genre and found each genre's share of said week's total listening, creating a panel of one row per genre per week. I exported this panel to a .csv file so that I had a jumping off point when working within the same week, and for visualizing and regressing my data. 

<img width="984" height="854" alt="image" src="https://github.com/user-attachments/assets/63c37786-6e0a-427d-8020-cb8d275fcfe8" />

#### Screenshot 3.1 (Loop Structure)

<img width="984" height="160" alt="image" src="https://github.com/user-attachments/assets/685f2486-d634-48b6-8466-91bff0a06e1c" />

#### Screenshot 3.2 (DataFrame Export)

<img width="448" height="430" alt="image" src="https://github.com/user-attachments/assets/d4d6228a-416c-4def-b9bf-7c39b2ca5e64" />

#### Screenshot 3.3 (CSV Export)

### Step 4: Merging with FRED Macro Data
Now that each of the genres had their play shares for each week, I needed to merge it with the macro data from FRED. I pulled each of the respective series using the "fredapi" package, converted them to one data frame that was separated by month and year. I then joined them to the previous data frame which contained artists and genres. I then filtered my data so that I could find data between 2005 and 2025, before saving everything to a .csv file. 

<img width="984" height="432" alt="image" src="https://github.com/user-attachments/assets/4004b628-e64e-4c54-8829-6a0a10bc4833" />

#### Screenshot 4.1 (FRED Series Pull)

<img width="982" height="625" alt="image" src="https://github.com/user-attachments/assets/51b97c36-aa71-4157-a732-ca1990a704dd" />

#### Screenshot 4.2 (Panel Merge and CSV Export)

<img width="867" height="704" alt="image" src="https://github.com/user-attachments/assets/b6ee78a0-cbd8-46b8-8a3c-90c283a3b1d0" />

#### Screenshot 4.3 (Final DataFrame)

### Step 5: Visualizing Listening Patterns Over Time
I built five plots (three of which are shown) in matplotlib/seaborn: a full genre breakdown over time, a cleaner version limited to the top 4 genres, total monthly listening volume, average monthly listening, and a genre-by-year heatmap. NBER recession periods were shaded on each time-series plot for visual reference.

<img width="687" height="493" alt="image" src="https://github.com/user-attachments/assets/956017c4-b398-4608-bdfc-294b0c09eca8" />

#### Screenshot 5.1 (Top 4 Genres' Play Share Over Time and Monthly Listening Activity, Subplots)

<img width="687" height="483" alt="image" src="https://github.com/user-attachments/assets/1a195d4f-eb41-462f-bbf9-6a1dfdaf3af2" />

#### Screenshot 5.2 (Genre-by-Year Heatmap)

### Step 6: Regression Analysis in R
In R, I ran two sets of linear models: one regressing total monthly plays (and log of total monthly plays) on the recession dummy, unemployment rate, and logged inflation/consumer sentiment. The second model regressed each genre's logged play share on the same macro variables plus logged total plays, run separately for genres of interest (Rock, Metal, Jazz, Hip Hop, Pop, Classical) rather than as a single pooled model with genre fixed effects, since separate regressions were more interpretable at this stage of the project. I did not run regressions for all of the genres, as they were relatively small portions of total listening throughout the entire series.

<img width="750" height="587" alt="image" src="https://github.com/user-attachments/assets/11ef461a-3d0a-48c2-9113-75470d76ed9f" />

#### Screenshot 6.1 (Top 4 Genres Regression)

<img width="401" height="556" alt="image" src="https://github.com/user-attachments/assets/7a25f382-3009-4758-ab69-92e85487dfee" />

#### Screenshot 6.2 (Total Plays Regression)

## Findings
Read these as descriptive results about **one individual's listening behavior,** not generalizable claims about how people listen to music during recessions. An individual's listening habits are largely affected by preferences and other factors that would be difficult to control for without survey data. The sample size here is one listener over roughly 20 years, which is enough to fit a regression but not enough to support population-level conclusions.
With that in mind, interpreting these results show interesting trends: total monthly plays dropped by nearly 60% during recession periods in this dataset, and inflation had a notably large negative effect on overall listening volume. During a recession, total monthly plays decrease by 234 on average, or roughly 59%, and with moderate statistical significance (10% level for total plays, 5% for logged total monthly plays). Increases in the U-3 Unemployment rate were also not very significant for the total plays regression, but became significant at the 5% level when plays were logged, showing a 1 percentage point increase in unemployment led to a 10% decrease in listening. A 1% increase in the UMCIH sentiment variable experienced the same pattern as the unemployment rate for significance: only signifiant in the logged play shares regression. Differing from the unemployment rate however, for a 1% increase in consumer sentiment, listening fell by 1.5%, with statistical significance at the 1% level. The most extreme finding was that a 1% increase in the PCE Price Index caused a fall of roughly 10.7 plays (0.01*1067), or 3.8%, and was significant at the 1% level for both the standard regression and the log of play shares.

Within the top 4 genres the only genre whose listening share increased by during recessions with statistical significance was metal, which increased by 72%; the coefficient here is extremely large, which I hypothesize reflects the listener's preferences. Despite being statistically insignificant, hip hop's play share saw the same relationship as metal with a 9% increase during recessions. Rock and jazz saw a smaller share of listening during recessions, with an 8% decrease and a 21% decrease respectively. Inflation had interesting effects on three of the four genres: rock decreased by 4.4%, metal by 3.3%, and jazz by 4.99% and all were significant at the 1% level. Additionally I found that for all four genres, the play share fell slightly (less than 1%) as total plays increased by 1%. Additionally a 1% increase in the UMICH variable decreased rock's play share by 2.6% and jazz's play share by .83% while metal and hip hop remained unaffected.

For the Special Cases regressions, I looked at the play shares for both pop and classical. To the point of pop, I had heard the term "recession pop" and wanted to see if pop was more reactive during recessions as a way to distract people from poor economic conditions. For the other special case, classical saw an odd spike after the COVID Pandemic and I was curious to see if they acted differently than the top 4 genres. I found that "recession pop" does not exist in this model, as there is no statistically significant effect on pop's play share nor was it significant for classical. Unemployment had a significant negative effect on pop's play share (roughly 8% decrease per one percentage point increase, p<0.01) while classical showed no significant relationship with unemployment. The same truth about music genre diversity was true for both classical and pop, and this showed that as total plays increased, the share of both of these genres decreased slightly (less than 1% for both). Inflation had a notable effect for classical's play share, and a 1% increase in the PCE price index corresponded to an increase of 3.5% on average (significant at the 5% level). Neither genre was effected by a change in consumer sentiment.

## Planned Extensions
As of 9/9/26 I am working on expanding this project with data that more accurately reflects what kind of music is popular. This includes finding a new data starting point and I am using a dataset that tracks historical weekly entries for the Billboard Hot 100. I will have to control for correlation between weeks, as popular songs don't go from spot #1 (most popular) to spot #100 (least popular, yet still top 100 in the country) within one week, and there is a more gradual decline in rank. In this continuation I would like to make the following changes, explanations included:

**Controlling for Mood/Song Content:** Putting an artist into one, singular, broad genre is a little shallow. Rock as a genre has some songs that feel happy and carefree, while others grapple with more somber themes. Even bands that would fit into the "Rock" macro-genre vary from song to song. An example of this can be seen in one popular band's, the Foo Fighters, second album titled "The Colour And The Shape". The second track on the album, "Monkey Wrench" contains lyrics and a beat that, to me, are very upbeat: speaking on themes on knowing one's value in a relationship, and ultimately deciding to prioritize oneself is a very uplifting message for the listener. On the same album, the eleventh track, and arguably the band's most famous song, contains lyrics expressing a fear about not finding anything in life that matches the level of enjoyment found in the present (we can assume this is an experience that involves a romantic partner or other meaningful relationship). Despite the more fearful, uncertain lyrics, the beat is energetic, providing clashing signals.
The conflict of themes found within *one* album from *one* band, shows how generalizing different artists into such broad genres may overshadow the content of the music, and provide a description that does not fit. I can control for this by analyzing both the Beats per minute (BPM) of the song, as well as keywords to make individual songs have more information than just "rock".

**Different Way to Measure Genres:** Relying on the collective Last.fm user base to provide tags that, are not only accurate but, fit into my genre buckets is unrealistic. In order to better analyze how music listening patterns change over genres it would make more sense to take genres that are already applied to artists. This would also avoid the problem of having to sift through tags that don't provide any insight to my project: many tags include jokes or factoids that stem from artists' fanbases and this code would simply bucket tags of this nature as "other" which diminishes the accuracy of my results.

**Compensate for Genre Size:** Within genres, there are tight knit communities that self proclaim their own descriptors for the kinds of music they prefer. For example, the genre of "Rock" can be broken down into subcategories of "Classic Rock" "Alternative Rock" "Punk Rock" "Yacht Rock" and so on. I hypothesize that a large genre such as "Classic Rock" would have a higher listen count than a much more specific genre such as "Shoegaze" and I would control for these by applying unit fixed effects which would account for consistent disparities in listening patterns. 

Overall this was an interesting exercise in mapping the listening habits of Last.fm's co-founder/creator, rj. The structure of the experiment is flexible enough so that I can scale this to include more global listening patterns. If I included more individuals listening data I would be able to discern more meaningful listening patterns and provide more actionable conclusions. **I am actively working on re-doing this project for my capstone undergraduate Economics Class, and I will be creating a repository for all of the code and explanations once I present it!** 

This project was completed independently for a data analytics coding course (Data Analytics (R and Python)(ECO 590)); the FRED-based econometric analysis was an extension beyond the course's core requirements, which focused primarily on the data collection, cleaning, and the mechanics of regression in R.

# How to attract app users

I am going to use AppleStore data on paid and free apps to look for what drives app use.


```python
def explore_data(dataset, start, end, rows_and_columns=False):
    dataset_slice = dataset[start:end]    
    for row in dataset_slice:
        print(row)
        print('\n') # adds a new (empty) line after each row

    if rows_and_columns:
        print('Number of rows:', len(dataset))
        print('Number of columns:', len(dataset[0]))
```

Open AppleStore and GoogleStore data


```python
open_file = open("AppleStore.csv")
from csv import reader
read_file = reader(open_file)
ios= list(read_file)
ios_header = ios[0]
ios = ios[1:]
```


```python
open_file = open("googleplaystore.csv")
from csv import reader
read_file = reader(open_file)
android = list(read_file)
android_header = android[0]
android = android[1:]
```


```python
print(ios_header)
```

    ['id', 'track_name', 'size_bytes', 'currency', 'price', 'rating_count_tot', 'rating_count_ver', 'user_rating', 'user_rating_ver', 'ver', 'cont_rating', 'prime_genre', 'sup_devices.num', 'ipadSc_urls.num', 'lang.num', 'vpp_lic']


More information on Apple app categories [here](https://www.kaggle.com/ramamet4/app-store-apple-data-set-10k-apps)



```python
explore_data(ios,  0, 4, True)
```

    ['284882215', 'Facebook', '389879808', 'USD', '0.0', '2974676', '212', '3.5', '3.5', '95.0', '4+', 'Social Networking', '37', '1', '29', '1']
    
    
    ['389801252', 'Instagram', '113954816', 'USD', '0.0', '2161558', '1289', '4.5', '4.0', '10.23', '12+', 'Photo & Video', '37', '0', '29', '1']
    
    
    ['529479190', 'Clash of Clans', '116476928', 'USD', '0.0', '2130805', '579', '4.5', '4.5', '9.24.12', '9+', 'Games', '38', '5', '18', '1']
    
    
    ['420009108', 'Temple Run', '65921024', 'USD', '0.0', '1724546', '3842', '4.5', '4.0', '1.6.2', '9+', 'Games', '40', '5', '1', '1']
    
    
    Number of rows: 7197
    Number of columns: 16



```python
print(android_header)
```

    ['App', 'Category', 'Rating', 'Reviews', 'Size', 'Installs', 'Type', 'Price', 'Content Rating', 'Genres', 'Last Updated', 'Current Ver', 'Android Ver']


More information on Android categories [here](https://www.kaggle.com/lava18/google-play-store-apps)


```python
explore_data(android, 0, 4, True)
```

    ['Photo Editor & Candy Camera & Grid & ScrapBook', 'ART_AND_DESIGN', '4.1', '159', '19M', '10,000+', 'Free', '0', 'Everyone', 'Art & Design', 'January 7, 2018', '1.0.0', '4.0.3 and up']
    
    
    ['Coloring book moana', 'ART_AND_DESIGN', '3.9', '967', '14M', '500,000+', 'Free', '0', 'Everyone', 'Art & Design;Pretend Play', 'January 15, 2018', '2.0.0', '4.0.3 and up']
    
    
    ['U Launcher Lite – FREE Live Cool Themes, Hide Apps', 'ART_AND_DESIGN', '4.7', '87510', '8.7M', '5,000,000+', 'Free', '0', 'Everyone', 'Art & Design', 'August 1, 2018', '1.2.4', '4.0.3 and up']
    
    
    ['Sketch - Draw & Paint', 'ART_AND_DESIGN', '4.5', '215644', '25M', '50,000,000+', 'Free', '0', 'Teen', 'Art & Design', 'June 8, 2018', 'Varies with device', '4.2 and up']
    
    
    Number of rows: 10841
    Number of columns: 13



```python

```




```python

```



## Data cleaning
There is a wrong entry for line [10472](https://www.kaggle.com/lava18/google-play-store-apps/discussion/66015)in the Google apps data. Rating is '19' which is out of range. I will delete this row.


```python
print(android[10472])
```

    ['Life Made WI-Fi Touchscreen Photo Frame', '1.9', '19', '3.0M', '1,000+', 'Free', '0', 'Everyone', '', 'February 11, 2018', '1.0.19', '4.0 and up']



```python
del android[10472]
```

Check for duplicate data.


```python
ios_unique = []
ios_duplicate = []

for app in ios:
    app_name = app[1]
    if app_name in ios_unique:
        ios_duplicate.append(app_name)
    else:
        ios_unique.append(app_name)
print("Number of duplicate apps in ios = ", len(ios_duplicate),'\n', "Examples of duplicates ",ios_duplicate[:15] )
        
```

    Number of duplicate apps in ios =  2 
     Examples of duplicates  ['Mannequin Challenge', 'VR Roller Coaster']



```python
android_unique = []
android_duplicate = []

for app in android:
    app_name = app[0]
    if app_name in android_unique:
        android_duplicate.append(app_name)
    else:
        android_unique.append(app_name)

print("Number of duplicate apps in Android = ", len(android_duplicate), '\n', "Examples of duplicate apps: ", android_duplicate[:15])
```

    Number of duplicate apps in Android =  1181 
     Examples of duplicate apps:  ['Quick PDF Scanner + OCR FREE', 'Box', 'Google My Business', 'ZOOM Cloud Meetings', 'join.me - Simple Meetings', 'Box', 'Zenefits', 'Google Ads', 'Google My Business', 'Slack', 'FreshBooks Classic', 'Insightly CRM', 'QuickBooks Accounting: Invoicing & Expenses', 'HipChat - Chat Built for Teams', 'Xero Accounting Software']


I will remove duplicate apps, keeping the newest data point that I can identify by highest amount of reviews.



```python
ios_reviews_max = {}

for app in ios:
    app_name = app[1]
    n_reviews = float(app[5])
    
    if app_name in ios_reviews_max and ios_reviews_max[app_name] < n_reviews:
        ios_reviews_max[app_name] = n_reviews
    elif app_name not in ios_reviews_max:
        ios_reviews_max[app_name] = n_reviews
print("Number of ios apps = ",len(ios_reviews_max))
        
        
    
```

    Number of ios apps =  7195



```python
android_reviews_max = {}

for app in android:
    app_name = app[0]
    n_reviews = float(app[3])
    
    if app_name in android_reviews_max and android_reviews_max[app_name] < n_reviews:
        android_reviews_max[app_name] = n_reviews
    elif app_name not in android_reviews_max:
        android_reviews_max[app_name]= n_reviews

print("Number of Android apps = ", len(android_reviews_max))
        
```

    Number of Android apps =  9659


 Remove rows with duplicate data from both sets.



```python
ios_clean = []
already_added = []

for app in ios:
    app_name = app[1]
    n_reviews = float(app[5])
    
    if n_reviews == ios_reviews_max[app_name] and app_name not in already_added:
        ios_clean.append(app)
        already_added.append(app_name)
print("We now have a total of",len(ios_clean), "ios apps.")
        
```

    We now have a total of 7195 ios apps.



```python
android_clean = []
already_added = []

for app in android:
    app_name = app[0]
    n_reviews = float(app[3])
    
    if n_reviews == android_reviews_max[app_name] and app_name not in already_added:
        android_clean.append(app)
        already_added.append(app_name)
print("We now have a total of", len(android_clean), "Android apps.")
        
```

    We now have a total of 9659 Android apps.


Only keep apps that are for English speakers. We can do this by definging a function that only keeping what we commonly use in an English text that are in the range 0 to 127, according to the [ASCII](https://en.wikipedia.org/wiki/ASCII). However, this will exlude an apps that contain emojis. We will allow an app to have up to three characters not within the range 0 -127 to allow for apps that may have emojis.


```python
def only_english(string):
    count = 0
    for char in string:
        if ord(char) > 127:
            count += 1
    if count >3:
        return False
    
    else:
        return True


ios_eng = []
android_eng = []

for app in ios_clean:
    app_name = app[1]
    
    if only_english(app_name):
        ios_eng.append(app)
        
for app in android_clean:
    app_name = app[0]
    count = 0
    if only_english(app_name):
        android_eng.append(app)

             
print('There are now', len(ios_eng), 'ios apps', 'and', len(android_eng), 'apps.')
             
```

    There are now 6181 ios apps and 9614 apps.


Now to only keep the free apps by removing all apps that have a fee.


```python
ios_free = []

for app in ios_eng:
    price = app[4]
    if price == '0.0':
        ios_free.append(app)
        
explore_data(ios_free, 0,0,True)        

```

    Number of rows: 3220
    Number of columns: 16



```python
android_free = []

for app in android_eng:
    price = app[7]
    if price == '0':
        android_free.append(app)
        
explore_data(android_free, 0, 0, True)
```

    Number of rows: 8864
    Number of columns: 13


Now we need to find the most popular apps: For ios, looks like the information we need will be:
- "ratingcounttot": User Rating counts (for all version)
- "user_rating" : Average User Rating value (for all version)
- "prime_genre": Primary Genre


For android:

- "Rating"
- "Reviews"
- "Genres"
- "Installs:


```python
def freq_table(dataset, index):
    table = {}
    total = 0
    
    for row in dataset:
        total += 1
        value = row[index]
        
        if value in table:
            table[value] += 1
    
        else:
            table[value] = 1 
       
    percents = {}
    for key in table:
        percentage = (table[key]/total) * 100
        percents[key] = percentage
        
    return percents
    
def display_table(dataset, index):
    table = freq_table(dataset, index)
    table_display = []
    for key in table:
        key_val_as_tuple = (table[key], key)
        table_display.append(key_val_as_tuple)

    table_sorted = sorted(table_display, reverse = True)
    for entry in table_sorted:
        print(entry[1], ':', entry[0])   
    
   
```

Displaying the frequencey table for "prime_genre" for ios apps.


```python
display_table(ios_free, 11)
```

    Games : 58.13664596273293
    Entertainment : 7.888198757763975
    Photo & Video : 4.968944099378882
    Education : 3.6645962732919255
    Social Networking : 3.291925465838509
    Shopping : 2.608695652173913
    Utilities : 2.515527950310559
    Sports : 2.142857142857143
    Music : 2.049689440993789
    Health & Fitness : 2.018633540372671
    Productivity : 1.7391304347826086
    Lifestyle : 1.5838509316770186
    News : 1.3354037267080745
    Travel : 1.2422360248447204
    Finance : 1.1180124223602486
    Weather : 0.8695652173913043
    Food & Drink : 0.8074534161490683
    Reference : 0.5590062111801243
    Business : 0.5279503105590062
    Book : 0.43478260869565216
    Navigation : 0.18633540372670807
    Medical : 0.18633540372670807
    Catalogs : 0.12422360248447205


For ios apps, games comprises the highest percentage, 58%. Entertainment is next with 7.9%. Education is, surprsingly, above social networking, but by a tony amount. 

Displaying the frequencey table for genre for Android apps.


```python
display_table(android_free,9 )
```

    Tools : 8.449909747292418
    Entertainment : 6.069494584837545
    Education : 5.347472924187725
    Business : 4.591606498194946
    Productivity : 3.892148014440433
    Lifestyle : 3.892148014440433
    Finance : 3.7003610108303246
    Medical : 3.531137184115524
    Sports : 3.463447653429603
    Personalization : 3.3167870036101084
    Communication : 3.2378158844765346
    Action : 3.1024368231046933
    Health & Fitness : 3.0798736462093865
    Photography : 2.944494584837545
    News & Magazines : 2.7978339350180503
    Social : 2.6624548736462095
    Travel & Local : 2.3240072202166067
    Shopping : 2.2450361010830324
    Books & Reference : 2.1435018050541514
    Simulation : 2.0419675090252705
    Dating : 1.861462093862816
    Arcade : 1.8501805054151623
    Video Players & Editors : 1.7712093862815883
    Casual : 1.7599277978339352
    Maps & Navigation : 1.3989169675090252
    Food & Drink : 1.2409747292418771
    Puzzle : 1.128158844765343
    Racing : 0.9927797833935018
    Role Playing : 0.9363718411552346
    Libraries & Demo : 0.9363718411552346
    Auto & Vehicles : 0.9250902527075812
    Strategy : 0.9138086642599278
    House & Home : 0.8235559566787004
    Weather : 0.8009927797833934
    Events : 0.7107400722021661
    Adventure : 0.6768953068592057
    Comics : 0.6092057761732852
    Beauty : 0.5979241877256317
    Art & Design : 0.5979241877256317
    Parenting : 0.4963898916967509
    Card : 0.45126353790613716
    Casino : 0.42870036101083037
    Trivia : 0.41741877256317694
    Educational;Education : 0.39485559566787
    Board : 0.3835740072202166
    Educational : 0.3722924187725632
    Education;Education : 0.33844765342960287
    Word : 0.2594765342960289
    Casual;Pretend Play : 0.236913357400722
    Music : 0.2030685920577617
    Racing;Action & Adventure : 0.16922382671480143
    Puzzle;Brain Games : 0.16922382671480143
    Entertainment;Music & Video : 0.16922382671480143
    Casual;Brain Games : 0.13537906137184114
    Casual;Action & Adventure : 0.13537906137184114
    Arcade;Action & Adventure : 0.12409747292418773
    Action;Action & Adventure : 0.10153429602888085
    Educational;Pretend Play : 0.09025270758122744
    Simulation;Action & Adventure : 0.078971119133574
    Parenting;Education : 0.078971119133574
    Entertainment;Brain Games : 0.078971119133574
    Board;Brain Games : 0.078971119133574
    Parenting;Music & Video : 0.06768953068592057
    Educational;Brain Games : 0.06768953068592057
    Casual;Creativity : 0.06768953068592057
    Art & Design;Creativity : 0.06768953068592057
    Education;Pretend Play : 0.056407942238267145
    Role Playing;Pretend Play : 0.04512635379061372
    Education;Creativity : 0.04512635379061372
    Role Playing;Action & Adventure : 0.033844765342960284
    Puzzle;Action & Adventure : 0.033844765342960284
    Entertainment;Creativity : 0.033844765342960284
    Entertainment;Action & Adventure : 0.033844765342960284
    Educational;Creativity : 0.033844765342960284
    Educational;Action & Adventure : 0.033844765342960284
    Education;Music & Video : 0.033844765342960284
    Education;Brain Games : 0.033844765342960284
    Education;Action & Adventure : 0.033844765342960284
    Adventure;Action & Adventure : 0.033844765342960284
    Video Players & Editors;Music & Video : 0.02256317689530686
    Sports;Action & Adventure : 0.02256317689530686
    Simulation;Pretend Play : 0.02256317689530686
    Puzzle;Creativity : 0.02256317689530686
    Music;Music & Video : 0.02256317689530686
    Entertainment;Pretend Play : 0.02256317689530686
    Casual;Education : 0.02256317689530686
    Board;Action & Adventure : 0.02256317689530686
    Video Players & Editors;Creativity : 0.01128158844765343
    Trivia;Education : 0.01128158844765343
    Travel & Local;Action & Adventure : 0.01128158844765343
    Tools;Education : 0.01128158844765343
    Strategy;Education : 0.01128158844765343
    Strategy;Creativity : 0.01128158844765343
    Strategy;Action & Adventure : 0.01128158844765343
    Simulation;Education : 0.01128158844765343
    Role Playing;Brain Games : 0.01128158844765343
    Racing;Pretend Play : 0.01128158844765343
    Puzzle;Education : 0.01128158844765343
    Parenting;Brain Games : 0.01128158844765343
    Music & Audio;Music & Video : 0.01128158844765343
    Lifestyle;Pretend Play : 0.01128158844765343
    Lifestyle;Education : 0.01128158844765343
    Health & Fitness;Education : 0.01128158844765343
    Health & Fitness;Action & Adventure : 0.01128158844765343
    Entertainment;Education : 0.01128158844765343
    Communication;Creativity : 0.01128158844765343
    Comics;Creativity : 0.01128158844765343
    Casual;Music & Video : 0.01128158844765343
    Card;Action & Adventure : 0.01128158844765343
    Books & Reference;Education : 0.01128158844765343
    Art & Design;Pretend Play : 0.01128158844765343
    Art & Design;Action & Adventure : 0.01128158844765343
    Arcade;Pretend Play : 0.01128158844765343
    Adventure;Education : 0.01128158844765343


There are way more categories for genre in Android withough enough overlap to compare with ios apps, perhaps looking at 'Category' might be better. 

Displaying the frequencey table for category for Android apps. I am not sure what 'Family' entails, and there seems to be a much more even spread throughout the categorie. Games is still one of the highest, however. 


```python
display_table(android, 1)
```

    FAMILY : 18.19188191881919
    GAME : 10.55350553505535
    TOOLS : 7.776752767527675
    MEDICAL : 4.271217712177122
    BUSINESS : 4.243542435424354
    PRODUCTIVITY : 3.911439114391144
    PERSONALIZATION : 3.616236162361624
    COMMUNICATION : 3.5701107011070112
    SPORTS : 3.5424354243542435
    LIFESTYLE : 3.5239852398523985
    FINANCE : 3.3763837638376386
    HEALTH_AND_FITNESS : 3.1457564575645756
    PHOTOGRAPHY : 3.0904059040590406
    SOCIAL : 2.7214022140221403
    NEWS_AND_MAGAZINES : 2.61070110701107
    SHOPPING : 2.3985239852398523
    TRAVEL_AND_LOCAL : 2.3800738007380073
    DATING : 2.158671586715867
    BOOKS_AND_REFERENCE : 2.1309963099630997
    VIDEO_PLAYERS : 1.6143911439114391
    EDUCATION : 1.4391143911439115
    ENTERTAINMENT : 1.3745387453874538
    MAPS_AND_NAVIGATION : 1.2638376383763839
    FOOD_AND_DRINK : 1.1715867158671587
    HOUSE_AND_HOME : 0.8118081180811807
    LIBRARIES_AND_DEMO : 0.7841328413284132
    AUTO_AND_VEHICLES : 0.7841328413284132
    WEATHER : 0.7564575645756457
    ART_AND_DESIGN : 0.5996309963099631
    EVENTS : 0.5904059040590406
    PARENTING : 0.5535055350553505
    COMICS : 0.5535055350553505
    BEAUTY : 0.488929889298893


Now we should find the most users by how many times an app has been installed. Android has this info under "Installs' but ios does not. We can guesstimate this by looking at how many people have rated the app per genre. I will use the frequency table I created using the category "prime_genre", which is index '11'.



```python
ios_genres = freq_table(ios_free, 11)

for genre in ios_genres:
    total = 0
    len_genre = 0
    for app in ios_free:
        genre_app = app[11]
        if genre_app == genre:
            n_ratings = float(app[5])
            total +=  n_ratings
            len_genre += 1
        
    average_n_ratings = total/len_genre
    print(genre, ':', average_n_ratings)


        
```

    Social Networking : 71548.34905660378
    Photo & Video : 28441.54375
    Games : 22812.92467948718
    Music : 57326.530303030304
    Reference : 74942.11111111111
    Health & Fitness : 23298.015384615384
    Weather : 52279.892857142855
    Utilities : 18684.456790123455
    Travel : 28243.8
    Shopping : 26919.690476190477
    News : 21248.023255813954
    Navigation : 86090.33333333333
    Lifestyle : 16485.764705882353
    Entertainment : 14029.830708661417
    Food & Drink : 33333.92307692308
    Sports : 23008.898550724636
    Book : 39758.5
    Finance : 31467.944444444445
    Education : 7003.983050847458
    Productivity : 21028.410714285714
    Business : 7491.117647058823
    Catalogs : 4004.0
    Medical : 612.0


Looks like navigation then reference have the most user ratings. I am sure this is because of google maps and Waze. Reference seems like a very wide category, not sure what reference means. Now I'll look at user installations for the Android data. This requires cleaning some of the data first, removing the plus signs and changing to a float.


```python
display_table(android_free, 5 )
```

    1,000,000+ : 15.726534296028879
    100,000+ : 11.552346570397113
    10,000,000+ : 10.548285198555957
    10,000+ : 10.198555956678701
    1,000+ : 8.393501805054152
    100+ : 6.915613718411552
    5,000,000+ : 6.825361010830325
    500,000+ : 5.561823104693141
    50,000+ : 4.7721119133574
    5,000+ : 4.512635379061372
    10+ : 3.5424187725631766
    500+ : 3.2490974729241873
    50,000,000+ : 2.3014440433213
    100,000,000+ : 2.1322202166064983
    50+ : 1.917870036101083
    5+ : 0.78971119133574
    1+ : 0.5076714801444043
    500,000,000+ : 0.2707581227436823
    1,000,000,000+ : 0.22563176895306858
    0+ : 0.04512635379061372
    0 : 0.01128158844765343



```python
android_cats = freq_table(android_free, 1)

for category in android_cats:
    total = 0
    len_category = 0
    for app in android_free:
        category_app = app[1]
        if category_app == category:
            n_installs = app[5]
            n_installs = n_installs.replace('+', '')
            n_installs = n_installs.replace(',', '')
            total += float(n_installs)
            len_category += 1
    average_installs = total/len_category
    print(category, ':', average_installs)

    

```

    ART_AND_DESIGN : 1986335.0877192982
    AUTO_AND_VEHICLES : 647317.8170731707
    BEAUTY : 513151.88679245283
    BOOKS_AND_REFERENCE : 8767811.894736841
    BUSINESS : 1712290.1474201474
    COMICS : 817657.2727272727
    COMMUNICATION : 38456119.167247385
    DATING : 854028.8303030303
    EDUCATION : 1833495.145631068
    ENTERTAINMENT : 11640705.88235294
    EVENTS : 253542.22222222222
    FINANCE : 1387692.475609756
    FOOD_AND_DRINK : 1924897.7363636363
    HEALTH_AND_FITNESS : 4188821.9853479853
    HOUSE_AND_HOME : 1331540.5616438356
    LIBRARIES_AND_DEMO : 638503.734939759
    LIFESTYLE : 1437816.2687861272
    GAME : 15588015.603248259
    FAMILY : 3695641.8198090694
    MEDICAL : 120550.61980830671
    SOCIAL : 23253652.127118643
    SHOPPING : 7036877.311557789
    PHOTOGRAPHY : 17840110.40229885
    SPORTS : 3638640.1428571427
    TRAVEL_AND_LOCAL : 13984077.710144928
    TOOLS : 10801391.298666667
    PERSONALIZATION : 5201482.6122448975
    PRODUCTIVITY : 16787331.344927534
    PARENTING : 542603.6206896552
    WEATHER : 5074486.197183099
    VIDEO_PLAYERS : 24727872.452830188
    NEWS_AND_MAGAZINES : 9549178.467741935
    MAPS_AND_NAVIGATION : 4056941.7741935486



```python

```

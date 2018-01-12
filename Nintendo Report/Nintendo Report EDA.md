Final Project
================
Tyler Mallon
11 January, 2018

Intial EDA
----------

I'll be starting out this EDA with a general exploration of this dataset. Once I have a decent understanding of distributions of variables, data types of the attributes and high level relationship between attributes I'll move into a more specific exploration of how Nintendo as a company is performing.

``` r
library(tidyverse)
library(readr)
library(ggthemes)
library(scales)
library(viridis)
library(ggiraph)
```

``` r
#importing dataset 
rm(list=ls(all=T))
df <- read_csv("Video_Games_Sales_as_at_22_Dec_2016.csv")
```

``` r
# Removing all rows with null values. Choosing not to try to do any imputation work on this dataset.
df.clean <- na.omit(df)
```

##### Note:

Dataset Drops from ~17000 to ~7000 observations

``` r
# Just looking to understand variable types/factor levels, etc.
summary(df.clean)
```

    ##      Name             Platform         Year_of_Release   
    ##  Length:6947        Length:6947        Length:6947       
    ##  Class :character   Class :character   Class :character  
    ##  Mode  :character   Mode  :character   Mode  :character  
    ##                                                          
    ##                                                          
    ##                                                          
    ##     Genre            Publisher            NA_Sales          EU_Sales      
    ##  Length:6947        Length:6947        Min.   : 0.0000   Min.   : 0.0000  
    ##  Class :character   Class :character   1st Qu.: 0.0600   1st Qu.: 0.0200  
    ##  Mode  :character   Mode  :character   Median : 0.1500   Median : 0.0600  
    ##                                        Mean   : 0.3928   Mean   : 0.2346  
    ##                                        3rd Qu.: 0.3900   3rd Qu.: 0.2100  
    ##                                        Max.   :41.3600   Max.   :28.9600  
    ##     JP_Sales        Other_Sales        Global_Sales      Critic_Score  
    ##  Min.   :0.00000   Min.   : 0.00000   Min.   : 0.0100   Min.   :13.00  
    ##  1st Qu.:0.00000   1st Qu.: 0.01000   1st Qu.: 0.1100   1st Qu.:62.00  
    ##  Median :0.00000   Median : 0.02000   Median : 0.2900   Median :72.00  
    ##  Mean   :0.06324   Mean   : 0.08219   Mean   : 0.7731   Mean   :70.26  
    ##  3rd Qu.:0.01000   3rd Qu.: 0.07000   3rd Qu.: 0.7500   3rd Qu.:80.00  
    ##  Max.   :6.50000   Max.   :10.57000   Max.   :82.5300   Max.   :98.00  
    ##   Critic_Count     User_Score          User_Count       Developer        
    ##  Min.   :  3.00   Length:6947        Min.   :    4.0   Length:6947       
    ##  1st Qu.: 14.00   Class :character   1st Qu.:   11.0   Class :character  
    ##  Median : 24.00   Mode  :character   Median :   27.0   Mode  :character  
    ##  Mean   : 28.87                      Mean   :  173.8                     
    ##  3rd Qu.: 39.00                      3rd Qu.:   88.0                     
    ##  Max.   :113.00                      Max.   :10665.0                     
    ##     Rating         
    ##  Length:6947       
    ##  Class :character  
    ##  Mode  :character  
    ##                    
    ##                    
    ## 

``` r
#Converting several features to factor types, converting other variable types to numeric types
df.clean <- df.clean %>%
  mutate_at(c("Platform", "Genre", "Publisher", "Developer", "Rating", "Year_of_Release"), funs(as.factor)) %>%
    mutate_at(c("User_Score", "Critic_Score", "Critic_Count", "User_Count"), funs(as.numeric))

#Further removing null values that weren't picked up after the first na removal. I think this happened because there were some values that were NA in the columns I converted to numeric columns and after conversion they converted to null so they can be picked up by na.omit

df.clean <- na.omit(df.clean)
df.clean <- filter(df.clean, Year_of_Release != "N/A")

# There are 3 observations of games that have a unique rating type. These are valid observations but for sake of simplicity in analysis, decided to only take observations with the normal 4 rating types.

df.clean <- filter(df.clean, Rating == "E10+" | Rating == "E" | Rating == "T" | Rating == "M")
```

##### Note:

Cleaning The Dataset Further. There are a couple levels of ratings that only have one or two observations and hardly ever appear. Also further removing N/A values

``` r
# First using nested ifelse for creating new factor variable. New variable classifies observations by the parent company that makes each platform. Second variable determines whether each game is exclusive to a particular platform or whether it was released across multiple platforms. 

df.clean <- df.clean %>%
  mutate(Company = ifelse(Platform == "Wii" | Platform == "DS" | Platform == "3DS" | Platform == "WiiU", "Nintendo", 
    ifelse(Platform == "X360" | Platform == "XB" | Platform == "XOne", "Microsoft", 
        ifelse(Platform == "DC", "Sega", 
            ifelse(Platform == "PC", "PC", "Sony")))), 
    
         Exclusive = !df.clean$Name %in% df.clean$Name[duplicated(df.clean$Name)]) %>%
    mutate(Exclusive = ifelse(Exclusive == TRUE, "Exclusive", "Cross-Platform")) %>%
  mutate_at(c("Company", "Exclusive"), funs(as.factor)) 

# Removing unused factor levels.

df.clean <- droplevels(df.clean)
```

##### Note:

Creating two new variables which are I am interested in. First one is combining the various consoles into their parent company. The second is determining whether a game is a multi-platform game or not.

``` r
summary(df.clean)
```

    ##      Name              Platform    Year_of_Release          Genre     
    ##  Length:6823        PS2    :1140   2008   : 592    Action      :1629  
    ##  Class :character   X360   : 858   2007   : 590    Sports      : 943  
    ##  Mode  :character   PS3    : 769   2005   : 561    Shooter     : 864  
    ##                     PC     : 651   2009   : 550    Role-Playing: 712  
    ##                     XB     : 564   2006   : 528    Racing      : 581  
    ##                     Wii    : 479   2003   : 498    Platform    : 403  
    ##                     (Other):2362   (Other):3504    (Other)     :1691  
    ##                        Publisher       NA_Sales          EU_Sales      
    ##  Electronic Arts            : 944   Min.   : 0.0000   Min.   : 0.0000  
    ##  Ubisoft                    : 496   1st Qu.: 0.0600   1st Qu.: 0.0200  
    ##  Activision                 : 492   Median : 0.1500   Median : 0.0600  
    ##  Sony Computer Entertainment: 315   Mean   : 0.3944   Mean   : 0.2361  
    ##  THQ                        : 307   3rd Qu.: 0.3900   3rd Qu.: 0.2100  
    ##  Nintendo                   : 291   Max.   :41.3600   Max.   :28.9600  
    ##  (Other)                    :3978                                      
    ##     JP_Sales        Other_Sales        Global_Sales      Critic_Score  
    ##  Min.   :0.00000   Min.   : 0.00000   Min.   : 0.0100   Min.   :13.00  
    ##  1st Qu.:0.00000   1st Qu.: 0.01000   1st Qu.: 0.1100   1st Qu.:62.00  
    ##  Median :0.00000   Median : 0.02000   Median : 0.2900   Median :72.00  
    ##  Mean   :0.06396   Mean   : 0.08268   Mean   : 0.7773   Mean   :70.26  
    ##  3rd Qu.:0.01000   3rd Qu.: 0.07000   3rd Qu.: 0.7500   3rd Qu.:80.00  
    ##  Max.   :6.50000   Max.   :10.57000   Max.   :82.5300   Max.   :98.00  
    ##                                                                        
    ##   Critic_Count      User_Score      User_Count     
    ##  Min.   :  3.00   Min.   :0.500   Min.   :    4.0  
    ##  1st Qu.: 14.00   1st Qu.:6.500   1st Qu.:   11.0  
    ##  Median : 25.00   Median :7.500   Median :   27.0  
    ##  Mean   : 28.93   Mean   :7.185   Mean   :  174.8  
    ##  3rd Qu.: 39.00   3rd Qu.:8.200   3rd Qu.:   89.0  
    ##  Max.   :113.00   Max.   :9.600   Max.   :10665.0  
    ##                                                    
    ##             Developer     Rating          Company    
    ##  EA Canada       : 149   E   :2082   Microsoft:1581  
    ##  EA Sports       : 142   E10+: 930   Nintendo :1187  
    ##  Capcom          : 126   M   :1433   PC       : 651  
    ##  Ubisoft         : 103   T   :2378   Sega     :  14  
    ##  Konami          :  95               Sony     :3390  
    ##  Ubisoft Montreal:  87                               
    ##  (Other)         :6121                               
    ##           Exclusive   
    ##  Cross-Platform:3857  
    ##  Exclusive     :2966  
    ##                       
    ##                       
    ##                       
    ##                       
    ## 

``` r
df.clean %>%
  ggplot(aes(x = Platform)) +
  geom_histogram(stat = "count")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-1.png)

``` r
df.clean %>%
  ggplot(aes(x = Year_of_Release)) +
  geom_histogram(stat = "count")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-2.png)

``` r
df.clean %>%
  ggplot(aes(x = Genre)) +
  geom_histogram(stat = "count")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-3.png)

``` r
df.clean %>%
  ggplot(aes(x = Rating)) +
  geom_histogram(stat = "count")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-4.png)

``` r
df.clean %>%
  ggplot(aes(x = NA_Sales)) +
  geom_histogram(bins = 50) +
  scale_x_continuous(limits = c(0,1.5))
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-5.png)

``` r
df.clean %>%
  ggplot(aes(x = EU_Sales)) +
  geom_histogram(bins = 50) +
  scale_x_continuous(limits = c(0,1.3))
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-6.png)

``` r
df.clean %>%
  ggplot(aes(x = JP_Sales)) +
  geom_histogram(bins = 50) +
  scale_x_continuous(limits = c(0,1))
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-7.png)

``` r
df.clean %>%
  ggplot(aes(x = Other_Sales)) +
  geom_histogram(bins = 50) +
  scale_x_continuous(limits = c(0,1))
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-8.png)

``` r
df.clean %>%
  ggplot(aes(x = Global_Sales)) +
  geom_histogram(bins = 50) +
  scale_x_continuous(limits = c(0,2))
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-9.png)

``` r
df.clean %>%
  ggplot(aes(x = Critic_Score)) +
  geom_histogram(bins = 42)
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-10.png)

``` r
df.clean %>%
  ggplot(aes(x = Critic_Count)) +
  geom_histogram(bins = 25)
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-11.png)

``` r
df.clean %>%
  ggplot(aes(x = User_Score)) +
  geom_histogram(bins = 47)
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-12.png)

``` r
df.clean %>%
  ggplot(aes(x = User_Count)) +
  geom_histogram(bins = 50) +
scale_x_continuous(limits = c(0,1000))
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-13.png)

``` r
df.clean %>%
  ggplot(aes(x = Company)) +
  geom_histogram(stat = "count")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-14.png)

``` r
df.clean %>%
  ggplot(aes(x = Exclusive)) +
  geom_histogram(stat = "count")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/univariate.histograms-15.png)

##### Observations:

1.  The review variables seem relatively normally distributed (although still skewed) but the sales values clearly follow some form of an exponential distribution. I'm interested in what the extreme high end of the sales distribution looks like and what games are there. Seeing what types of games are in the extreme end of the tail is a very interesting question to me.

2.  This dataset primarily has games from the mid to late 2000's and as such, the primary consoles that are going to be represented here are the PS2/PS3 for Sony, Wii/DS for Nintendo & XBOX/XBOX360 For Microsoft since this is the time period that these consoles were operating in

``` r
# Removing Sega from the analysis as they haven't been a relevant player in the console space for the past 15-20 years

df.clean <- filter(df.clean, Company != "Sega")
df.clean <- droplevels(df.clean)
levels(df.clean$Rating) <- c("E", "E10+", "T", "M")
```

##### Note:

Removing Sega from the consideration as they are no longer a competitor in the console market of today as they shifted to an entirely software focused business. Historically they used to be, but given the histogram of "Year\_of\_release" I can see that most of the observations fall outside of the years that Sega was producing their own consoles.

``` r
df.clean %>%
  #Going to take log value of global sales since distribution is exponential. Multiplying by 100 to remove decimal values from observations since min(df.clean$Global_Sales) = 0.01
  ggplot(aes(x = Critic_Score, y = log(Global_Sales*100))) + 
  geom_point() +
  facet_grid(Company ~ Exclusive)
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.1-1.png)

``` r
df.clean %>%
  #Same as previous graph but with user_score instead of critic_score
  ggplot(aes(x = User_Score, y = log(Global_Sales*100))) +
  geom_jitter() +
  facet_grid(Company ~ Exclusive)
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.1-2.png)

##### Observations:

1.  Critic scores follow more linearly with global sales than user scores do. This makes sense to me as I would guess consumers inform their buying decisions based off of what critics say about the game. This also seems to be true across most sub-categories, such as Company & Exclusivitity

``` r
df.clean %>%
  group_by(Platform) %>%
  summarise_at("Global_Sales", median) %>%
  ggplot(aes(x = Platform, y = Global_Sales, fill = Global_Sales)) +
    geom_bar(stat = "identity") +
    coord_flip()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.2-1.png)

``` r
df.clean %>%
  group_by(Genre) %>%
  summarise_at("Global_Sales", median) %>%
  ggplot(aes(x = Genre, y = Global_Sales, fill = Global_Sales)) +
    geom_bar(stat = "identity") +
    coord_flip()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.2-2.png)

``` r
df.clean %>%
  group_by(Company) %>%
  summarise_at("Global_Sales", median) %>%
  ggplot(aes(x = Company, y = Global_Sales, fill = Global_Sales)) +
    geom_bar(stat = "identity") +
    coord_flip()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.2-3.png)

``` r
df.clean %>%
  group_by(Exclusive) %>%
  summarise_at("Global_Sales", median) %>%
  ggplot(aes(x = Exclusive, y = Global_Sales, fill = Global_Sales)) +
    geom_bar(stat = "identity") +
    coord_flip()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.2-4.png)

``` r
df.clean %>%
  group_by(Platform) %>%
  summarise_at("Global_Sales", sum) %>%
  ggplot(aes(x = Platform, y = Global_Sales, fill = Global_Sales)) +
    geom_bar(stat = "identity") +
    coord_flip()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.2-5.png)

``` r
df.clean %>%
  group_by(Genre) %>%
  summarise_at("Global_Sales", sum) %>%
  ggplot(aes(x = Genre, y = Global_Sales, fill = Global_Sales)) +
    geom_bar(stat = "identity") +
    coord_flip()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.2-6.png)

``` r
df.clean %>%
  group_by(Company) %>%
  summarise_at("Global_Sales", sum) %>%
  ggplot(aes(x = Company, y = Global_Sales, fill = Global_Sales)) +
    geom_bar(stat = "identity") +
    coord_flip()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.2-7.png)

``` r
df.clean %>%
  group_by(Exclusive) %>%
  summarise_at("Global_Sales", sum) %>%
  ggplot(aes(x = Exclusive, y = Global_Sales, fill = Global_Sales)) +
    geom_bar(stat = "identity") +
    coord_flip()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.2-8.png)

##### Observations:

1.  Some of the median value bar charts are interesting: Cross-Platform games typically sell more than exclusives. Sony typically is selling the best. Action games have by far sold the most but on average they don't perform as well as sports games, platformers, or some other categories.

2.  PC is incredibly low but I think that's just an unusual sample that we have. I'm guessing this dataset isn't account for the multitude of indie games that are regularly released. I also don't think it's accounting for games that monetize in consistent manner such as Free to Play games and Subscription based games. For this reason, I don't think the PC category is particularly well represented so I'm probably going to exclude it from the further analysis and keep this just about the Big 3 console players (Microsoft, Sony, Nintendo)

``` r
df.clean %>%
  group_by(Platform) %>%
  # Taking mean values since the critic_score distribution was not heavily skewed
  summarise_at("Critic_Score", mean) %>%
  ggplot(aes(x = Platform, y = Critic_Score, fill = Critic_Score)) +
    geom_bar(stat = "identity") +
    coord_flip()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.3-1.png)

``` r
df.clean %>%
  group_by(Genre) %>%
  # Taking mean values since the critic_score distribution was not heavily skewed
  summarise_at("Critic_Score", mean) %>%
  ggplot(aes(x = Genre, y = Critic_Score, fill = Critic_Score)) +
    geom_bar(stat = "identity") +
    coord_flip()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.3-2.png)

``` r
df.clean %>%
  ggplot(aes(x = Critic_Count, y = Critic_Score)) +
  geom_jitter()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.3-3.png)

``` r
df.clean %>%
  ggplot(aes(x = User_Count, y = User_Score)) +
  geom_jitter()
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/graphical.exploration.3-4.png)

##### Observations:

1.  Interesting that the Wii has one of the lowest average critical review scores when earlier we saw it was in top 5 selling consoles. It must have really popular games that were not critically very favorable.

2.  Not surprisingly, the more critics that review a game, the more likely it will have a better score since the better the game is, the more buzz it probably generates. User reviews are less reliable and are generally less well correlated. This relationship is clearly heteroskedastic and it would be important to remember this in any type of modeling effort

Nintendo Analysis
-----------------

I think I now have a decent grasp on this dataset and am ready to move into a more specific analysis of what is going on with Nintendo as a company. This portion of the EDA is going to be almost entirely graphical as it is my intention to now develop a series of visualizations that will help explore the relationships of gaming companies and their video game sales.

``` r
# As I mentioned a bit above, my data sample is very poorly representative of PC in general and so I'm going to exclude it from further analysis.
df.clean <- df.clean %>%
  filter(Company != "PC")
```

``` r
df.clean %>%
  group_by(Company) %>%
  summarise_at("Global_Sales", sum) %>%
  ggplot(aes(x = Company, y = Global_Sales, fill = Company)) +
  geom_bar(stat = "identity") +
scale_fill_viridis(option = "C", discrete = T, end = 0.85, direction = -1) +
  # cleaning up the labeling on the axes with the line below
  scale_y_continuous(labels = dollar_format(prefix = "$", suffix = "M")) +
  theme_minimal() +
  theme(text = element_text(family = "Helvetica"), axis.title = element_text(size = "16", face = "bold"), legend.title = element_text(size = "12", face = "bold"), plot.title = element_text(size = "18", face = "bold"),
        axis.title.x = element_blank(), strip.text = element_text(size = "11")) +
  labs(y = "Global Sales", title = "Sony Crushes Nintendo's Global Video Game Sales", subtitle = "Sony Has Sold Slightly More Than Double What Nintendo Has")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/nintendo.1.1-1.png)

##### Observations:

1.  Sony is significantly outperforming the competition, selling nearly double what Microsoft & Nintendo have. I'd like to now break down these figures across a number of different factors to better understand why Sony is so dominant.

``` r
df.clean %>%
group_by(Company, Exclusive) %>%
    summarise_at("Global_Sales", sum) %>%
    ggplot(aes(x = Company, y = Global_Sales, fill = Exclusive)) +
      geom_bar(stat = "identity") +
scale_fill_viridis(option = "C", discrete = T, end = 0.85, direction = -1) +
  scale_y_continuous(labels = dollar_format(prefix = "$", suffix = "M")) +
  theme_minimal() +
  theme(text = element_text(family = "Helvetica"), axis.title = element_text(size = "16", face = "bold"), legend.title = element_text(size = "12", face = "bold"), plot.title = element_text(size = "18", face = "bold"),
        axis.title.x = element_blank(), strip.text = element_text(size = "11")) +
  labs(y = "Global Sales", title = "Nintendo & Sony Are About Even On Exclusives", subtitle = "Nintendo Massively Underperforms On Cross-Platform Games Relative To Competitors")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/nintendo.1.2-1.png)

##### Observations:

1.  Nintendo's sales are almost all exclusives. They have very little Cross-Platform representation. Microsoft seems to be the reverse of Nintendo, and Sony seems to be the best of both worlds.

``` r
df.clean %>%
group_by(Company, Rating) %>%
    summarise_at("Global_Sales", sum) %>%
    ggplot(aes(x = Company, y = Global_Sales, fill = Rating)) +
      geom_bar(stat = "identity") +
scale_fill_viridis(option = "C", discrete = T, end = 0.85, direction = -1) +
  scale_y_continuous(labels = dollar_format(prefix = "$", suffix = "M")) +
  theme_minimal() +
  theme(text = element_text(family = "Helvetica"), axis.title = element_text(size = "16", face = "bold"), legend.title = element_text(size = "12", face = "bold"), plot.title = element_text(size = "18", face = "bold"),
        axis.title.x = element_blank(), strip.text = element_text(size = "11")) +
  labs(y = "Global Sales", title = "Nintendo Ignores More Mature Audiences", subtitle = "Nintendo's Sales Are Almost Entirely Comprised Of Family Friendly Games")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/nintendo.1.3-1.png)

##### Observations:

1.  Nintendo's Sales are almost all E or E10+. This implies that they could be targeting a specific audience, have the best success with family friendly ratings, or both.

2.  Microsoft seems to be the opposite of Nintendo, with their games being concentrated in the T or M ratings. Sony again seems to have found success with both audiences.

``` r
df.clean %>%
group_by(Company, Exclusive, Rating) %>%
    summarise(Global_Sales = sum(Global_Sales)) %>%
    ggplot(aes(x = Company, y = Global_Sales, fill = Rating)) +
      geom_bar(stat = "identity")  +
        facet_grid(. ~ Exclusive) +
scale_fill_viridis(option = "C", discrete = T, end = 0.85, direction = -1) +
  scale_y_continuous(labels = dollar_format(prefix = "$", suffix = "M")) +
  theme_minimal() +
  theme(text = element_text(family = "Helvetica"), axis.title = element_text(size = "16", face = "bold"), legend.title = element_text(size = "12", face = "bold"), plot.title = element_text(size = "18", face = "bold"),
        axis.title.x = element_blank(), strip.text = element_text(size = "11")) +
  labs(y = "Global Sales", title = "Nintendo's Problem: The Best Selling Cross-Platform\nGames Are For Mature Audiences", subtitle = "Nintendo Has Largely Ignored T & M Rated Cross-Platform Games While Competitors Embraced Them")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/nintendo%202.1-1.png)

##### Observations:

1.  T & M rated cross platform games are a huge part of the market which Microsoft and Sony have clearly capitalized on. It appears that Microsoft's sales are primarily comprised of Cross-Platform games, but the distribution or ratings seems to be fairly consistent across both Cross-Platform & Exclusive games.
2.  Sony has phenomenal sales for both Cross-Platform games as well as Exclusive games. Sony's Cross-Platform & Exclusive games are also both fairly equally distributed across the different ratings, with having a slightly higher concentration of M games in their exclusives compared to Cross-Platform games.

3.  Nintendo has a very clear focus area of what they have sold, with their E rated exlclusive games making up more than all other categories combined.

``` r
# Creating a new data frame that filters the data frame to contain only the top 100 games in terms of global sales
top.sales <- df.clean %>%
  top_n(100, Global_Sales)

# Alternate method for creating a new data frame that ranks games by the top 100 critical scores. This method is required because top_n doesn't handle ties particularly well.
top.critic <- df.clean[order(df.clean$Critic_Score, df.clean$Critic_Count, decreasing = T),]
top.critic <- top.critic[1:100,]
```

``` r
top.sales %>%
group_by(Company, Exclusive, Rating) %>%
    summarise_at("Global_Sales", sum) %>%
    ggplot(aes(x = Company, y = Global_Sales, fill = Rating)) +
      geom_bar(stat = "identity") +
        facet_grid(. ~ Exclusive) +
scale_fill_viridis(option = "C", discrete = T, end = 0.85, direction = -1) +
  scale_y_continuous(labels = dollar_format(prefix = "$", suffix = "M")) +
  theme_minimal() +
  theme(text = element_text(family = "Helvetica"), axis.title = element_text(size = "16", face = "bold"), legend.title = element_text(size = "12", face = "bold"), plot.title = element_text(size = "18", face = "bold"),
        axis.title.x = element_blank(), strip.text = element_text(size = "11")) +
  labs(y = "Global Sales", title = "Nintendo's Flagship Games Are A Powerful Force", subtitle = "Nintendo's Family Friendly Exclusive Games Sell Better Than Any Other Single Group Of Games")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/nintendo.3.1-1.png)

##### Observations:

1.  Nintendo's exclusive games have dominated the best sellers list. Although Sony has been much more successful on aggregate, Nintendo seems to have produced some of the best selling games of all time. Moreover, almost all of Nintendo's best selling games have been exclusive titles that are rated E.

``` r
top.critic %>%
group_by(Company, Exclusive, Rating) %>%
    summarise_at("Global_Sales", sum) %>%
    ggplot(aes(x = Company, y = Global_Sales, fill = Rating)) +
      geom_bar(stat = "identity") +
        facet_grid(. ~ Exclusive) +
scale_fill_viridis(option = "C", discrete = T, end = 0.85, direction = -1) +
  scale_y_continuous(labels = dollar_format(prefix = "$", suffix = "M")) +
  theme_minimal() +
  theme(text = element_text(family = "Helvetica"), axis.title = element_text(size = "16", face = "bold"), legend.title = element_text(size = "12", face = "bold"), plot.title = element_text(size = "18", face = "bold"),
        axis.title.x = element_blank(), strip.text = element_text(size = "11")) +
  labs(y = "Global Sales", title = "The Top-Rated Games", subtitle = "Mature Cross-Platform Games are critically acclaimed and sell well")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/nintendo.4.1-1.png)

##### Observations:

1.  Relative to the best selling games in our data set, the sales values of the top rated games look very different. Nintendo is clearly far less successful at producing games that are critically acclaimed in addition to selling well. Moreover, we can see that there is a clear type of game that seems to be both critically acclaimed and sell well: Mature Cross-Platform games that have a T or M rating.

``` r
top.sales %>%
  mutate(Competitor = ifelse(Company == "Nintendo", "Nintendo", "Competitor")) %>%
    ggplot(aes(x = Critic_Score, y = Global_Sales, color = Competitor)) +
      geom_jitter(stat = "identity") +
scale_color_viridis(option = "C", discrete = T, end = 0.85, direction = -1) +
  scale_y_continuous(labels = dollar_format(prefix = "$", suffix = "M")) +
  theme_minimal() +
  theme(text = element_text(family = "Helvetica"), axis.title = element_text(size = "16", face = "bold"), legend.title = element_blank(), plot.title = element_text(size = "18", face = "bold")) +
  labs(x = "Critical Review Metascore", y = "Global Sales", title = "The Top 100 Selling Games: Nintendo's Domain", subtitle = "Nothing Even Comes Close To Some Of Nintendo's Biggest Hits")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/nintendo.5.1-1.png)

##### Observations:

1.  Our data set contains one clear outlier making up over $80 M of Nintendo's sales, but it is clear that the very top of the distribution of best selling games is dominated by Nintendo entries. Of the 13 games to have sold over $20 M, 10 of them are from Nintendo. Neither Sony nor Microsoft are even close to surpassing Nintendo's record on the highest selling game in this data set.

``` r
top.critic %>%
  mutate(Competitor = ifelse(Company == "Nintendo", "Nintendo", "Competitor")) %>%
    ggplot(aes(x = Critic_Score, y = Global_Sales, color = Competitor)) +
      geom_jitter(stat = "identity") +
scale_color_viridis(option = "C", discrete = T, end = 0.85, direction = -1) +
  scale_y_continuous(labels = dollar_format(prefix = "$", suffix = "M")) +
  theme_minimal() +
  theme(text = element_text(family = "Helvetica"), axis.title = element_text(size = "16", face = "bold"), legend.title = element_blank(), plot.title = element_text(size = "18", face = "bold")) +
  labs(x = "Critical Review Metascore", y = "Global Sales", title = "The Top 100 Critically Reviewed Games:\nNintendo Struggles", subtitle = "Nintendo Has Far Fewer Entries On The Top 100 Critically Reviewed Games Than Competitors")
```

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20EDA%20Images/nintendo.5.2-1.png)

##### Observations:

1.  Looking at the the sales performance of the top 100 critically reviewed games is a much different story than looking at the top 100 best selling games. It seems that Nintendo's competitors have done a much better job at translating critically acclaimed games into top sellers than Nintendo has. For a variety of reasons, it seems that Nintendo's monster hits were not that critically favorable.

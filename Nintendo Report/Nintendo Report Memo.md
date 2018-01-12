What's Holding Back Nintendo?
================
Tyler Mallon
10 January, 2018

### Introduction

For the past 30 years, Nintendo has been synonymous with home console gaming. They have created some of gaming's most iconic characters such as Mario & Luigi, Pikachu, and Donkey Kong. They have outlasted many of their former competitors such as Sega & Atari. They have continually innovated new and interesting ways to play games; without them the gaming industry would be a shadow of its current self. Unfortunately, Nintendo are no longer the market leaders they once were. So what went wrong? Even though historically Nintendo has brought great innovation to the gaming industry, they have become complacent and have not adapted well to the rapid evolution of the gaming industry. To better illustrate the challenges that Nintedo faces, I have downloaded [this dataset](https://www.kaggle.com/gregorut/videogamesales) from Kaggle containing sales information and review scores for thousands of games over the past 20 years. It is my intention for this analysis to serve as a wake up call for Nintendo.

------------------------------------------------------------------------

### Market Orientation

The market leader could be defined by a number of metrics, but I am choosing to go with total global sales for games. One that is often chosen is the number of consoles sold by each company, as games sold are impacted by the number of people that own a console. Although it's not normalized for number of consoles sold, global sales for games does have two primary benefits:

1.  Many consoles today are more than just machines for playing games. They are utilized for other forms of media and can be bought and used only for these purposes instead of actually playing games
2.  Compared to number of consoles sold, it is easier to break down global sales of games into categories that allow us to understand why consumers prefer one company over another.

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20Images/nintendo.1.1-1.png)

***Figure 1:*** Sony is crushing the competition. They have sold nearly double the amount of games that Nintendo and Microsoft have. The key to understanding where Nintendo is struggling is to break these totals down by rating and exclusivity.

------------------------------------------------------------------------

### Market Breakdown

##### Sales By Exclusivity

One of the factors in a consumer's decision of which console to buy are the exclusive games that are only playable on that console. Breaking down total sales by games that are exclusive vs. cross-platform will help show the relative importance that exclusive games have for each company.

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20Images/nintendo.1.2-1.png)

***Figure 2:*** When it comes to sales of exclusive games, Nintendo is actually performing about the same as Sony. This shows that Nintendo's main problem area is cross-platform games. Conversely, Microsoft has the opposite problem of Nintendo. The fact that Sony is performing well in both areas plays a large factor in their market dominance.

------------------------------------------------------------------------

##### Sales By Rating

Rating is a valuable way of breaking down games sold because it captures two different pieces of information.

1.  Rating can be a valuable way of understanding the targeted demographic for a particular game.
2.  Rating is a proxy variable for the genre of a particular game. Mature, action oriented games such as shooters are often classified as T or M, while family-friendly games such as sports games or puzzle games are often classified as E or E10+

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20Images/nintendo.1.3-1.png)

***Figure 3:*** Again, we see that Nintendo's game sales are comprised by an asymmetric distribution among Ratings. Most of their sales comes from E or E10+ rated games, while Microsoft's games are primarily the more mature T or M rated games. Sony, to their credit, performs well in both family friendly rated games as well as mature rated games.

------------------------------------------------------------------------

##### Sales By Rating & Exclusivity

Looking at Sales by Rating & Exclusivity individually provides valuable insight as to where Nintendo is performing well and where they are not. However, looking at these variables together will help us better understand the relationship between the two variables and ultimately give us a better idea of why Nintendo is struggling.

![](https://github.com/wat3rm3ll0n/VIZ/blob/master/Nintendo%20Report/Nintendo%20Images/nintendo%202.1-1.png)

***Figure 4:*** There are three imporant things to point out from this graph.

1.  Almost all of Nintendo's exclusive games are rated E or E10+. This shows that what Nintendo is trying to sell in order to set itself apart from the competition are family friendly games.
2.  The best selling cross-platform games are T or M rated games, an area that Nintendo has historically struggled in.
3.  Lastly, mature, cross-platform games are a large portion of the overall market. Whether it is a strategic decision or not, Nintendo clearly cannot afford to perform as poorly in this part of the market as they have been.

------------------------------------------------------------------------

### Conclusion

The big takeaway I have is that Nintendo is too reliant on their exclusive games. Family friendly games such as platformers, sports games or puzzle games are their bread and butter. They have tried to have these exclusive game offerings support their company, and to some degree, this has been successful. Unfortunately though, there is a large part of the market that wants to play more mature rated games, such as shooters or action games. I believe that Nintendo is in control of their own destiny though. Many of these mature games are cross-platform games developed by third parties, which means that if Nintendo wanted to move in a direction to better support these types of games, they should be able to do so. What the market has shown is that being able to support both types of games is a recipe for success, and it is why Sony is the market leader.

---
layout: post
title: Crypto, python, dash - Year on year price changes
date: 2018-02-13 09:00:17 +0800
category: python
tags: python dash crypto dashboard
comments: TRUE
---
<a href="http://www.coins-alarm.com:8050/cpc"><img src="https://raw.githubusercontent.com/fathomson/fathomson.github.io/master/assets/img/dashboard_cpc.PNG" width="300px"></a>
<br>
The prices of crypto have always been subject to severe price fluctuations, which is logical since there is no centralized authority to stabilize the prices. You might be familiar with the <a href='https://bravenewcoin.com/assets/Uploads/four-stages-chart.jpg'>hype cycle</a> which perfectly describes the evolvement stages of emerging technologies. Not surprisingly, the crypto prices can be fit to the hype cycle too. A lot has been written about that already and this post will no go there but only focus on the developed dashboard.

#### Why this dashboard?
Two reasons, one for the community and one personal. The community reason would be to give perspective to the new folks joining the crypto space. The main consideration for most people to start with crypto is to get rich quickly and happened to invest during the peak moments of the hype cycle. The personal reason is to get improve my python knowledge and get acquainted with dash and its dashboarding possibilities.

#### How does it work?
###### Usage
Once the dashboard is loaded: select a crypto, a start and end date and the rate you want to use in the calculations. After a change the dashboard will automatically reload. The list of crypto's is pulled from coinmarketcap and so is the historical data. The last option you can select is the rate or price, which can be Open, Close, High, Low. As mentioned earlier, the dashboard will put the price fluctuations in perspective and could come in handy when making investment decisions.
At moment of writing I have hosted the dashboard somewhere for you to use. I cannot guaranty that it will be there forever and I recommend that if you plan to use it frequently to host it somewhere yourself.

####### Technical
Upon start the dashboard collects the daily price data of the selected crypto from coinmarketcap. The data is then aggregated on a weekly level and the price change compared to 1 Jan of that specific year is calculated. The next step is to format the data in such a way that plotly can visualize it and that it can be shown on the dashboard. If you are interested in the python code have a look at the github repo, link below.

##### Useful links
[Project source code](https://github.com/fathomson/CryptoPriceChanges)  
[Report issues](https://github.com/fathomson/CryptoPriceChanges/issues)  
[Questions/contact](https://www.linkedin.com/in/fathomson/)  

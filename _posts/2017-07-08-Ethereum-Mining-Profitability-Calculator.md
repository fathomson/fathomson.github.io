---
layout: post
title: Ether mining profitability calculator
---

### Introduction

A long time ago I purchased a Butterfly Labs rig to mine Bitcoin. During that time I lived in a student house for which the rent included energy costs. This seemed like a perfect deal and a quick calculation showed that I would earn my investment back in just a matter of weeks. However, at that point the difficulty started to increase significantly and each day that passed shifted my break even date with two days. For this you do not need to be an economist to see that the break even date would never come. So after a while I decided to sell the mining rig and just invest it in bitcoin.

You will probably think, why is this guy talking about bitcoin while the title clearly states ether, well that is because of the following: since a few months ether is on a daily bases in the news. Some articles are about the recent price increase or about the opportunities and risks and even some about mining. And the articles about mining and cloud mining triggered me to build a reliable ether mining profitability calculator. As with the bitcoin story there are mining profitability calculators available, however these work with fixed or simple difficulty values. And with the current growth in net hash rate basically all these profitability calculators provide a too rosy image. That is why I build a profitability calculator which, based on the predicted net hash rate and daily ether to miner rewards, provides a realistic view of your investment returns.    

This post describes the steps I took to create the calculator and how you can use it.

### Data collection
To be able to predict the net hash rate and daily ether to miners I needed data. I ran a quick search on google but could not find a recent full dump of the ether chain, which made me decide to collect it myself. To do this I downloaded geth and started the ether chain synchronization process. It took a while to get the almost 4M blocks but once I had them I could start the data collection. With  `geth --rpc --rpccorsdomain localhost` I started a local instance of the rpc client which enabled me to very quickly communicate with the chain.

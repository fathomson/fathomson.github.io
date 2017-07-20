---
layout: post
title: R, ethereum, shinyapps - Profitability dashboard part 1 of 2
date: 2017-07-20 18:30:13 +0800
category: feedR
tags: R shinyapps ethereum dashboard
comments: TRUE
---
<a href="https://ethereum.shinyapps.io/dashboard/"><img src="https://raw.githubusercontent.com/fathomson/fathomson.github.io/master/assets/img/dashboard.PNG" width="300px"></a>
<br>
The cryptocurrency Ethereum now reaches the news headlines on a daily basis. It has attracted a lot of new miners who often seek a way to earn a quick buck, but how profitable is the mining of this digital currency? To find out I walk you thought the steps I took to develop a profitability calculator dashboard. It consists out of two parts, the first part focusses on the data collection and preparation and in the second part I will handle the dashboard development.

#### Collect data
The first step is to get the full ether block chain to collect the data from, so I downloaded and installed [geth](https://geth.ethereum.org/downloads/).
{% highlight r %}
#create account
geth account new

#sync chain and start rpc
geth --rpc --rpccorsdomain localhost --cache=1024
{% endhighlight %}

This took quite a while and you could speed it up by downloading a so-called backup file of the chain which are available online. While this process runs you can already start collecting block data. Do note that the full size of the chain currently is about 60GB and counting.

{% highlight r linenos %}
# json rpc api functions.
# thanks to: https://github.com/BSDStudios/ethr
# json-rpc documentation: https://github.com/ethereum/wiki/wiki/JSON-RPC

# Returns the number of most recent block.
eth_blockNumber <- function(rpc_address = "http://localhost:8545") {
  post_body <- list(jsonrpc = "2.0", method = "eth_blockNumber", params = "", id = 83)
  post_return <- httr::POST(url = rpc_address, body = post_body, encode = "json")
  post_content <- httr::content(post_return, as = "parsed")
  block_number <- post_content$result
  return(block_number)
}

# Returns information about a block by block number.
eth_getBlockByNumber <-function(block_number, full_list, rpc_address = "http://localhost:8545") {
  block_number <- as.character(block_number)
  full_list <- as.logical(full_list)
  body <- list(jsonrpc = "2.0", method = "eth_getBlockByNumber", params = list(block_number, full_list), id = 1)
  block_return <- httr::POST(url = rpc_address, body = body, encode = "json")
  block_dat <- httr::content(block_return)$result
  return(block_dat)
}

# Returns the receipt of a transaction by transaction hash.
eth_getTransactionReceipt <- function(transaction_hash, rpc_address = "http://localhost:8545") {
  body <- list(jsonrpc = "2.0", method = "eth_getTransactionReceipt",  params = list(transaction_hash), id = 1)
  TransReceipt_return <- httr::POST(url = rpc_address, body = body, encode = "json")
  TransReceipt <- httr::content(TransReceipt_return)$result
  return(TransReceipt)
}

# Returns information about a uncle of a block by number and uncle index position.
eth_getUncleByBlockNumberAndIndex <- function(block_number, index, rpc_address = "http://localhost:8545") {
  body <- list(jsonrpc = "2.0", method = "eth_getUncleByBlockNumberAndIndex",  params = list(block_number, index), id = 1)
  UncleBlock_return <- httr::POST(url = rpc_address, body = body, encode = "json")
  UncleBlock <- httr::content(UncleBlock_return)$result
  return(UncleBlock)
}
{% endhighlight %}

#### Transform data
The data as received from the functions defined above is not directly suitable. It needs to be transformed and enriched in order to be of use. The functions below help us transforming the data.
{% highlight r linenos %}
# hex -> numeric
getNumber <- function(hex){
  class(hex) <- "numeric"
  return(as.numeric(sprintf("%.0f",hex)))
}

# numeric -> hex
getHex <-  function(number){
  return(paste0("0x",sprintf("%x",number)))
}

# Enrich block data with: uncle and transaction details.
enrichETHblock <- function(hexBlock){

  # default block reward, amount transactionfees and total ether of transactions in block.
  blockReward <- 5
  txFees <- 0
  ethValue <- 0

  # sum all transactionfees
  for(transaction in hexBlock$transactions){    
    ethValue <- ethValue + getNumber(transaction$value)/1e18
    tr <- eth_getTransactionReceipt(transaction$hash)
    if(is.null(tr))
      next
    txFees <- txFees + getNumber(tr$gasUsed) * (getNumber(transaction$gasPrice)/1e18)
  }

  # sum total uncle rewards
  uncleETH <- 0
  for(i in 0:1){
    uncle <- eth_getUncleByBlockNumberAndIndex(hexBlock$number,getHex(i))
    if(!is.null(uncle))
      uncleETH <- uncleETH + ((8-(getNumber(hexBlock$number) - getNumber(uncle$number)))/8) * blockReward
  }

  # miner reward for included uncle
  minerRewardUncle <- length(hexBlock$uncles) * 1/32 * blockReward

  # wrap in dataframe
  block <- data.frame(number = getNumber(hexBlock$number),
                      timestamp = as.POSIXct(getNumber(hexBlock$timestamp), origin="1970-01-01"),
                      difficulty=  getNumber(hexBlock$difficulty),
                      txCount = length(hexBlock$transactions),
                      txEther = ethValue,
                      txFees = txFees,
                      uncleETH = uncleETH,
                      minerUncleReward = minerRewardUncle,
                      totalETHreward = blockReward + txFees + minerRewardUncle + uncleETH)
  return(block)
}

{% endhighlight %}

#### Save data
Ok, now we have collected the data and we can transform it into the format we want. The next step is saving the transformed data for our future analysis. I have chosen for a local instance of SQL to write the data to. If you do not have an instance of SQL on your machine download it and create a database add a table.  I have named my table eth_blocks and it can be created with the following statement:
{% highlight sql %}
CREATE TABLE eth_blocks(
	[number] [int] NULL,
	[timestamp] [datetime] NULL,
	[difficulty] [bigint] NULL,
	[txCount] [int] NULL,
	[txEther] [decimal](24, 16) NULL,
	[txFees] [decimal](24, 16) NULL,
	[uncleETH] [decimal](24, 16) NULL,
	[minerUncleReward] [decimal](24, 16) NULL,
	[totalETHreward] [decimal](24, 16) NULL
)
{% endhighlight %}

Once the table is created we need to be able to write the data from R into SQL. The are two beautiful R libraries available to help us achieve that namely:
{% highlight r %}
library(RODBC)
library(RODBCext) # parameterizes queries

# Define an ODBC datasource and you can easily connect to your database
dbHandle <-odbcConnect("[database-name]")
{% endhighlight %}

Once everything is set up we can start the script that actually collects the data, transforms it and finally saves it into the database.

{% highlight r linenos%}
# script start time
start.time <- Sys.time()

# create database handle ODBS data source
dbHandle <-odbcConnect("ethereum")

# get the latest block in database
maxBlockInDB <- sqlQuery(dbHandle, "select top 1 number from eth_blocks order by number desc")
maxBlockInDB <- as.numeric(maxBlockInDB)

# when no block is found, assume that there are no block in the db yet
if(is.na(maxBlockInDB[1]))
  maxBlockInDB <- 0

# start block is latest block in db + 1
startBlock <- maxBlockInDB + 1

# stop block is last block in chain
stopBlock <- getNumber(eth_blockNumber())

# blocks that will be added in this session. 1 when stopblock and startblok are equal.
print(paste0("Starting update operation, blocks to add: : ",stopBlock -maxBlockInDB ))

# get new blocks from chain and write to database    
for (i in startBlock:stopBlock){
  # get block i from chain
  block <- eth_getBlockByNumber(getHex(i), TRUE)
  if(is.null(block))
    break

  # convert to database format and save in database.
  rBlock <- enrichETHblock(block)
  RODBCext::sqlExecute(dbHandle, "INSERT INTO eth_blocks VALUES (?,?,?,?,?,?,?,?,?)", rBlock)

  # once every 500 blocks show status messuge
  if(i %% 500 == 1){
    end.time <- Sys.time()
    time.taken <- end.time - start.time
    print(paste0("Current block : ",i," Operation took:", as.numeric(time.taken)   ))
    start.time <- Sys.time()
  }
}
# when done show status message again.
print(paste0("Done! added: : ",stopBlock -maxBlockInDB," blocks" ))

# when done retrieve all blocks from the database
allBlocks <- sqlQuery(dbHandle, "select * from eth_blocks order by number asc")

# aggregate block data
dailyDifficulty <- allBlocks %>%
  dplyr::mutate(toTimestamp = dplyr::lead(timestamp))   %>%
  dplyr::mutate(duration = as.numeric(difftime(toTimestamp,timestamp, units = "secs")))   %>%
  dplyr::mutate(weigthedDifficulty = difficulty*duration)   %>%
  dplyr::group_by(Day = as.Date(format(as.Date(timestamp, "%d/%m/%Y")))) %>%
  dplyr::summarise(netHash = sum(weigthedDifficulty)/sum(duration) / mean(duration),
                   avgBlocktime = mean(duration),
                   ethFromBlocks = n()*5,
                   ethFromUnclesForMiner = sum(uncleETH),
                   ethFromUnclesForIncluder = sum(minerUncleReward),
                   ethFromtxFees = sum(txFees),
                   ethTotal = sum(ethFromBlocks,ethFromUnclesForMiner,ethFromUnclesForIncluder,ethFromtxFees))

# RStudio filedir
# write to dashboard dir
dir <- dirname(rstudioapi::getActiveDocumentContext()$path)
saveRDS(dailyDifficulty, paste0(dir, "/dashboard/data/daily.Rda"))

# clode db connection
odbcCloseAll()
{% endhighlight %}

As you notice in the end I save the aggregated data file (daily.Rda) in a separate folder. This data file will be used for our dashboard which I discuss in part 2.

##### Useful links
[Project source code](https://github.com/fathomson/Ether-mining-profitability-calculator)  
[Report issues](https://github.com/fathomson/Ether-mining-profitability-calculator/issues)  
[Questions/contact](https://www.linkedin.com/in/fathomson/)  

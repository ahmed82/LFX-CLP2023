# Transactions Per Second (TPS)
- Number of transactions per second through the network (Fabric network). The time for one transaction from the **submit to the endorser** to the **final commit to the ledger** is taken as time considered for the transaction. 

TPS = (Total Successful Transactions) / (Total time taken)

### Factors influencing the TPS
- Number of Endorsing nodes (as execution of chaincode will be more parallelized), causing increasing TPS.
- off loading the bulky data to IFPS network instead of storing them in Fabric network
- Indexing using **CouchDB / LevelDB** (The state database).
- Increasing the number of endorser channel.



### Principle of TPS Calculation for Caliper 
> TPS = (Total Successful Transactions) / (Total time taken)

> Total Time = Last committing time  - First submitting time




### Our Work
Currently, what I have is basically, submitting the envelop (signed read/write sets to the ordering service directly), and evaluating the time taken to finally propose the blocks by ordering the transactions to the block. This we can apparantly say as the TPS through the ordering service not for the entire network. 

As our comparision is just based on the change of one variable, which is the ordering service, so this should be a good benchmark for comparision. 


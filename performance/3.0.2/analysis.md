#3.0.2 Analysis (***document not yet completed)

##Test Scenarios

* All tests were run on Centos 6 VMs
* Tests were run for couchbase server on **4 Core** and **8 core** machines with **4GB** and **8GB** RAM
* Tests were run with 100% resident ratio and with DGM (about 50% resident ratio)
* Replication, replication with views, replication with xdcr, replication with xdcr and views and with multiple number of external clients tested with front end load
* Build used 3.0.2-1644-rel

##Conclusions

###Baseline Tests
* Good until 12k load
* with views
* with xdcr
* with xdcr and views

###Effect of external clients
* 1 client to 5 clients ok
* Initial replication queue build up
* Double build up
* steady state and memcached threading
* Why after about 100ms
* Effect on front end load

###4 core vs 8 core
* 8 core better but similar behaviour characteristics
* with more cores we can have more load

###4GB RAM vs 8GB RAM
* 8 GB ram we can have more clients without enduring the pain of initial replica queue build up due to backfill
* 

###DGM vs Non DGM
* Similar results with baseline tests
* But cannot have more ext clients as it suffers from initial replica queue build up due to backfill

##Comparision with Sherlock
* No initial replica queue build up due to backfill. We have backfill snooze

##Further Improvements


##All test results
* [3.0.2 results](overview.md)
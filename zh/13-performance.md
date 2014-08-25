---
layout: post
title: 
---

Performance

The data used in the metrics is that of what would be considered a fairly standard application in MongoDB that Mongoid could manage.

    1,000,000 documents in the main collection
    10,000 embedded documents on a root document (1-n)
    10,000 embedded documents on a root document (1-1)
    100,000 referenced documents to another collection (1-n)
    100,000 referenced documents to another collection (1-1)
    10,000 referenced documents to another collection (n-n)

* Performance metrics done on a 2.93 GHz i7 iMac / 8GB RAM, OSX 10.7, Mongoid 2.2.0

* The script to run the performance metrics can be found here.
Notes

Many to many relations are not recommended for over 10,000 documents when using MRI due to the garbage collector taking over 90% of the run time when calling #build or #create. This is due to the large array appending occuring in these operations.
	When performing appends to one to many relations we execute in batch (appending all at once instead of one at a time) due to the slow performance of MongoDB's $push atomic operator.
Operation 	Time 	Ops/sec
root (1,000,000 operations)
Model#create 	417.93 	2,392
Model#all.each 	40.87 	24,476
Model#find 	0.001 	
Model#save 	487.93 	2,049
Model#update_attribute 	339.54 	2,945
embedded 1-n (10,000 operations)
relation#build 	2.253 	4,438
relation#clear 	1.230 	8,130
relation#create 	4.899 	2,041
relation#count 	0.011 	
relation#delete_all 	1.377 	7,262
relation#push (batch) 	3.496 	2,860
relation#each 	0.027 	370,370
relation#find 	0.044 	
relation#delete 	0.045 	
embedded 1-1 (10,000 operations)
relation#= 	3.902 	2,562
relational 1-n (100,000 operations)
relation#build 	18.521 	5,399
relation#clear 	4.085 	24,479
relation#create 	45.464 	2,199
relation#count 	0.051 	
relation#delete_all 	5.596 	17,869
relation#push (batch) 	34.551 	2,894
relation#each 	0.055 	1,818,181
relation#find 	0.020 	
relation#delete 	0.410 	
relational 1-1 (100,000 operations)
relation#= 	54.991 	1,818
relational n-n (10,000 operations)
relation#build 	1.325 	7,547
relation#clear 	0.350 	28,571
relation#count 	0.001 	
relation#delete_all 	0.001 	10,000,000
relation#push (batch) 	2.628 	3,805
relation#each 	0.005 	2,000,000
relation#find 	0.010 	
relation#delete 	0.060 	
eager loading 1-1 (10,000 docs)
Model#each (without eager load) 	5.617 	1,780
Model#includes (with eager load) 	2.966 	3,371
eager loading 1-n (10,000 docs)
Model.all#each (without eager load) 	4.300 	2,325
Model#includes (with eager load) 	2.094 	4,775


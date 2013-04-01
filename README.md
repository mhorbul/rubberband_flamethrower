Rubberband Flamethrower
=======================
Rubberband Flamethrower is a collection of scripts for dealing with faked Elastic Search data. 

The main script "inserter.rb" is a script for rapidly inserting test data into Elastic Search. It inserts fake "tweet" type objects into a "twitter" index on a local Elastic Search server at localhost:9200.  It runs in an infinite loop until you stop it. 

The script "retriever.rb" also runs in a constant loop, doing a search on the tweets type in the twitter index for all objects within the date range between 2 and 3 minutes ago and reports the number of objects found.  This can be used to easily approximate the maximum speed obtainable for inserting to a local Elastic Search index for a given AWS box size.

Pre-Requisites
=======================
This script has been written with ruby 1.9.3 in mind. It requires two gems: httparty and active_support. Httparty is used to send commands to the Elastic Search server.  Active Support is used for the JSON library in core_ext.

To run the inserter script:
=======================
The script "inserter.rb" will create and store objects with a message, username, and postDate to a local Elastic Search index called "twitter" for the type "tweet". The message is composed of 6 to 16 random words and capped at 140 characters.

	cd /path/to/repository/rubberband_flamethrower
	ruby inserter.rb

There are several random word lists in the "words" folder which come from SCOWL http://wordlist.sourceforge.net/scowl-readme

To run the retriever:
=======================
The retriever script is set up to run in a continuous loop and report the number of objects inserted into Elastic Search during the specified time period. It defaults to a span of one minute (really one minute and one second) that occurred between two and three minutes ago.

	cd /path/to/repository/rubberband_flamethrower
	ruby retriever.rb

Benchmarks
=======================

Running one AWS Small Instance. I installed and started Elastic Search and then cloned this project onto the box and started the script running in a screen session.

Running only the insert script during the time period being sampled by the retriever with it set to the default one minute (and one second) span.

A few samples from the run show that we are inserting anywhere from 6,000 to 13,000 documents in Elastic Search.

	{"query":{"range":{"postDate":{"from":"20130401T20:50:24","to":"20130401T20:51:24"}}}}
	11325
	{"query":{"range":{"postDate":{"from":"20130401T20:50:25","to":"20130401T20:51:25"}}}}
	11317
	{"query":{"range":{"postDate":{"from":"20130401T21:08:01","to":"20130401T21:09:01"}}}}
	6365
	{"query":{"range":{"postDate":{"from":"20130401T21:08:02","to":"20130401T21:09:02"}}}}
	6328
	{"query":{"range":{"postDate":{"from":"20130401T21:12:33","to":"20130401T21:13:33"}}}}
	12689
	{"query":{"range":{"postDate":{"from":"20130401T21:12:34","to":"20130401T21:13:34"}}}}
	12684

Running the insert script and retriever script during the time period being sampled by the retriever for a short period did not seem to greatly affect the number, though running the retriever for a long duration did greatly degrade the performance of the inserts and increased load average on the box.

Running the retriever on a sample period of an hour (and one second) instead of one minute (and one second) we still see a fairly large variance in the number being inserted per hour and so far have only seen it increasing in time. This is a very small sample and too brief and investigation to draw any real conclusions yet.

	{"query":{"range":{"postDate":{"from":"20130401T20:18:43","to":"20130401T21:18:43"}}}}
	377978
	{"query":{"range":{"postDate":{"from":"20130401T20:18:44","to":"20130401T21:18:44"}}}}
	378122
	{"query":{"range":{"postDate":{"from":"20130401T20:30:59","to":"20130401T21:30:59"}}}}
	462394
	{"query":{"range":{"postDate":{"from":"20130401T20:31:00","to":"20130401T21:31:00"}}}}
	462503
	{"query":{"range":{"postDate":{"from":"20130401T20:34:19","to":"20130401T21:34:19"}}}}
	485449
	{"query":{"range":{"postDate":{"from":"20130401T20:34:20","to":"20130401T21:34:20"}}}}
	485577
	{"query":{"range":{"postDate":{"from":"20130401T20:40:10","to":"20130401T21:40:10"}}}}
	529912
	{"query":{"range":{"postDate":{"from":"20130401T20:40:11","to":"20130401T21:40:11"}}}}
	529981


TO DO/Things to Look Into/Notes
=======================
 - Does message length varying account for some of the variance in insert speed? How much?
 - What role does the size of the pool of words used to construct the random sentences play in the time it takes to insert into the index?
 - What happens when we add more nodes to the Elastic Search cluster?
 - As the script runs longer and longer on the limited word set does it become increasingly faster or slower at indexing new content? To put it another way, does having a lot that already uses the same words make it faster or does it slow down because it now matches it up to that much more information.
 - How realistic is it to use Elastic Search nodes in the same manner as MySQL master/slave relationships where all selects are done from other nodes and all inserts on a master? Is this just common sense good practice, why didn't I see this mentioned anywhere?
 - These scripts need to be run from inside the rubberband_flamethrower directory because the script is using a relative link to the random words files.
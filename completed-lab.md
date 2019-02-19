# Lab: PIG and Hive

## Start your cluster

1. Start an Amazon Elastic MapReduce (EMR) Cluster using Quickstart with the following setup:
	* In *General Configuration*
		*  Give the cluster a name that is meaningful to you
		*  Un-check *Logging*
		*  Make sure that *Cluster* is selected, NOT *Step execution*
	*  In *Software configuration*
		*  Select `emr-5.20.0` Release from the drop-down
		*  Select the first option under Applications
	*  In *Hardware configuration*
		*  Select `m4.large` as instance types 
		*  Enter `3` for number of instances (1 master and 2 core nodes)
	* In *Security and access*
	* 	Select your correct EC2 keypair or you will not be able to connect to the cluster
	*  Leave everything else the same
	*  Click **Create Cluster**, and wait...

2. Once the cluster is up and running and in "waiting" state, ssh into the master node: `ssh hadoop@[[master-node-dns-name]]`

3. Install git on the master node: `sudo yum install -y git`

3. Clone this repository to the master node. Note: since this is a public repository you do can use the `http` GitHub URL: `git clone https://github.com/bigdatateaching/lab-pig-hive.git`

4. Change directory into the lab: `cd lab-pig-hive` 

## PIG Exercises

1. Look at the contents of the file `pigdemo.txt` using the `cat` command. This is a pretty small file. This file is in the **remote** filesystem.

	```
	[hadoop@ip-172-31-2-208 lab-pig-hive]$ cat pigdemo.txt
	SD	Rich
	NV	Barry
	CO	George
	CA	Ulf
	IL	Danielle
	OH	Tom
	CA	manish
	CA	Brian
	CO	Mark
	```

2. Start the Grunt Shell: `pig`

3. You can run HDFS commands from the Grunt Shell:
	- `grunt> ls`
	- Make a directory within the cluster HDFS called lab06: `mkdir lab`
	- Copy the `pigdemo.txt` file from the **local** filesystem to HDFS: `copyFromLocal pigdemo.txt lab/`
	- Check to make sure the file was copied: 
	
	```
	grunt> ls lab/
	hdfs://ip-172-31-2-208.ec2.internal:8020/user/hadoop/pig-hive-lab/pigdemo.txt<r 2>	80
	```
	
1. Define the `employees` relation and load data from the `pigdemo.txt` file (from HDFS - remember, you just copied the file from remote filesystem to HDFS), using a schema with field names state and name:

	```
	grunt> employees = LOAD 'pig-hive-lab/pigdemo.txt' AS (state, name);
	```

4. Use the describe command to see what the `employees` relation looks like:
	
	```
	grunt> describe employees;
	employees: {state: bytearray,name: bytearray}
	```

4. Use `DUMP` to see the contents of the employees relation:

	```
	grunt> DUMP employees
	... lots of text from the spawned MapReduce job ...
	(SD,Rich)
	(NV,Barry)
	(CO,George)
	(CA,Ulf)
	(IL,Danielle)
	(OH,Tom)
	(CA,manish)
	(CA,Brian)
	(CO,Mark)
	```
	
4. Create a new relation called `ca_only` and filter the `employees` relation to get only the records where the state is `CA` (California):

	```
	grunt> ca_only = FILTER employees BY (state=='CA');
	681844 [main] WARN  org.apache.pig.newplan.BaseOperatorPlan  - Encountered Warning IMPLICIT_CAST_TO_CHARARRAY 1 time(s).
	18/02/26 17:23:01 WARN newplan.BaseOperatorPlan: Encountered Warning IMPLICIT_CAST_TO_CHARARRAY 1 time(s).
	grunt>
	```

4. See the contents of `ca_only`. The output is still tuples but only the records that match the filter.

	```
	grunt> DUMP ca_only
	... lots of text from the spawned MapReduce job ...
	(CA,Ulf)
	(CA,manish)
	(CA,Brian)
	```
	
4. Create a new relation called `emp_group` where records are grouped by state:

	```
	grunt> emp_group = GROUP employees BY state;
	1186375 [main] WARN  org.apache.pig.newplan.BaseOperatorPlan  - Encountered Warning IMPLICIT_CAST_TO_CHARARRAY 1 time(s).
	18/02/26 17:31:25 WARN newplan.BaseOperatorPlan: Encountered Warning IMPLICIT_CAST_TO_CHARARRAY 1 time(s).
	```


4. Describe `emp_group`. Bags represent groups in PIG. A Bag is an unordered collection of tuples;

	```
	grunt> describe emp_group
	emp_group: {group: bytearray,employees: {(state: bytearray,name: bytearray)}}
	```

4. See the contents of the `emp_group`:

	```
	grunt> DUMP emp_group
	... lots of text from the spawned MapReduce job ...
	(CA,{(CA,Ulf),(CA,manish),(CA,Brian)})
	(CO,{(CO,George),(CO,Mark)})
	(IL,{(IL,Danielle)})
	(NV,{(NV,Barry)})
	(OH,{(OH,Tom)})
	(SD,{(SD,Rich)})
	```
4. Use the `STORE` command to write the `emp_group` file back to HDFS (or S3). Notice that the name given is that of a directory, not of a file

	```
	grunt> STORE emp_group INTO 'emp_group';
	... lots of text from the spawned MapReduce job ...
	Input(s):
	Successfully read 9 records (80 bytes) from: "hdfs://ip-172-31-2-208.ec2.internal:8020/user/hadoop/pig-hive-lab/pigdemo.txt"
	
	Output(s):
	Successfully stored 6 records (128 bytes) in: "hdfs://ip-172-31-2-208.ec2.internal:8020/user/hadoop/emp_group"
	```
	
4. To see all the relations that you have created: `aliases`


## Use PIG to generate a dataset for Hive

1. Exit the Grunt Shell if you haven't already: `grunt> quit;`
2. Make sure you are within the lab directory. You can check by using the Linux command `pwd` (present working directory). You should be in `/home/hadoop/lab-pig-hive`, if not change to it.
3. Unzip the White House visits dataset: `unzip whitehouse_visits.zip`. This creates a file called `whitehouse_visits.txt`
4. Create a directory within HDFS: `hadoop fs -mkdir whitehouse`
5. Copy the `whitehouse_visits.txt` file from the local filesystem into HDFS
	`hadoop fs -put whitehouse_visits.txt whitehouse/visits.txt` (Note the file has been renamed within HDFS)
5. Explore the contents of the Pig file `wh_visits.pig`. This is a pre-written Pig script that loads the `visits.txt` and extracts certain fields, filters, and writes the output to a location in HDFS: `cat wh_visits.pig`
6. Run the Pig script: `pig wh_visits.pig`
7. You should have a set of new files in the `hive_wh_visits` directory inside HDFS:

	```
	[hadoop@ip-172-31-2-208 lab-pig-hive]$ hadoop fs -ls
	Found 3 items
	drwxr-xr-x   - hadoop hadoop          0 2018-02-26 20:29 hive_wh_visits
	drwxr-xr-x   - hadoop hadoop          0 2018-02-26 18:14 lab
	drwxr-xr-x   - hadoop hadoop          0 2018-02-26 20:28 whitehouse
	```
	
8. Explore the contents of the results file/files:

	```
	[hadoop@ip-172-31-2-208 lab-pig-hive]$ hadoop fs -cat hive_wh_visits/part-v000-o000-r-00000 | head
	BUCKLEY	SUMMER	10/12/2010 14:48	10/12/2010 14:45	WH
	CLOONEY	GEORGE	10/12/2010 14:47	10/12/2010 14:45	WH
	PRENDERGAST	JOHN	10/12/2010 14:48	10/12/2010 14:45	WH
	LANIER	JAZMIN		10/13/2010 13:00	WH	BILL SIGNING/
	MAYNARD	ELIZABETH	10/13/2010 12:34	10/13/2010 13:00	WH	BILL SIGNING/
	MAYNARD	GREGORY	10/13/2010 12:35	10/13/2010 13:00	WH	BILL SIGNING/
	MAYNARD	JOANNE	10/13/2010 12:35	10/13/2010 13:00	WH	BILL SIGNING/
	MAYNARD	KATHERINE	10/13/2010 12:34	10/13/2010 13:00	WH	BILL SIGNING/
	MAYNARD	PHILIP	10/13/2010 12:35	10/13/2010 13:00	WH	BILL SIGNING/
	MOHAN	EDWARD	10/13/2010 12:37	10/13/2010 13:00	WH	BILL SIGNING/
	cat: Unable to write to output stream.
	```
8. You will use this file as an input to Hive in the next exercise
	
## Hive Exercises

1. Start the hive console:

	```
	[hadoop@ip-172-31-2-208 lab-pig-hive]$ hive
	
	Logging initialized using configuration in file:/etc/hive/conf.dist/hive-log4j2.properties Async: false
	hive>
	```
	
2. Create an **External Table** from the files created from the PIG output called `wh_visits`:

	```
	create external table wh_visits (
	lname string, 
	fname string,
	time_of_arrival string, 
	appt_scheduled_time string,
	meeting_location string,
	info_comment string
	) 
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY '\t' 
	LOCATION  '/user/hadoop/hive_wh_visits/';
	```

3. Write a SQL query to count the number of records in the `wh_visits` table:

	```
	select count(*) from wh_visits;
	```

4. Shot the first 20 records of the `wh_visits` table: 

	```
	select * from wh_visits limit 20;
	```

5. Write a SQL query that gives you the count of comments for records where the comment is not empty.

6. Once you figure out the correct SQL statement, create a CSV file in HDFS with the results of the query from the previous step. The syntax to use is the following:

	```
	INSERT OVERWRITE DIRECTORY '[[hdfs/s3 output directory]]'
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ',' 
	select ...;
	```
	Where:
	- `[[hdfs/s3 output directory]]` is the name of a **directory** to be created, where the results files from the MapReduce process will be placed.
	- `select ...` is the query





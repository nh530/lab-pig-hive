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

### Part 1

1. Look at the contents of the file `pigdemo.txt` using the `cat` command. This is a pretty small file. This file is in the **remote** filesystem.
2. Start the Grunt Shell: `pig`
3. You can run HDFS commands from the Grunt Shell: `grunt> ls`
4. From within the grunt console, make a directory within the cluster HDFS called lab
5. From within the grunt console, copy the `pigdemo.txt` file from the **local** filesystem to HDFS: `copyFromLocal pigdemo.txt lab/`
1. Define the `employees` relation and load data from the `pigdemo.txt` file (from HDFS - remember, you just copied the file from remote filesystem to HDFS), using a schema defining field names "state" and "name"
4. Use the describe command to see what the `employees` relation looks like:
4. Use `DUMP` to see the contents of the employees relation:
4. Create a new relation called `ca_only` and filter the `employees` relation to get only the records where the state is `CA` (California):
4. See the contents of `ca_only`. 
4. Create a new relation called `emp_group` where records are grouped by state:
4. Look at the structure of the relation `emp_group` by using the `describe` command
4. See the contents of the `emp_group` on screen
4. Save the `emp_group` to disk (S3 and/or HDFS) using the `STORE` command
4. To see all the relations that you have created: `aliases`


### Part 2: Use PIG to generate a dataset for Hive

1. Exit the Grunt Shell 
2. Make sure you are within the lab directory. You can check by using the Linux command `pwd` (present working directory)
3. Unzip the White House visits dataset: 
4. Create a directory called `whitehouse` in HDFS
5. Copy the `whitehouse_visits.txt` file from the local filesystem into HDFS in the `whitehouse` directory and rename it `visits.txt`
5. Explore the contents of the Pig file `wh_visits.pig`. This is a pre-written Pig script that loads the `visits.txt` and extracts certain fields, filters, and writes the output to a location in HDFS
6. Run the `wh_visits.pig` script **from the command line**
7. After the Pig script runs. you should have a set of new files in the in HDFS
8. Explore the contents of the results file/files:
	
## Hive Exercises

Your input file for this exercise is the Pig output created by the `wh_visits.pig`.

1. Start the hive console: `hadoop@ip-172-31-2-208 lab-pig-hive]$ hive`
2. Create an **External Table** from the files created from the PIG output called `wh_visits`, and defint the table with the following field names and types:

* lname (string) 
* fname (string)
* time\_of\_arrival (string) 
* appt_scheduled_time (string)
* meeting_location (string)
* info_comment (string)

3. Write a SQL query to count the number of records in the `wh_visits` table:
4. Show the first 20 records of the `wh_visits` table: 
5. Write a SQL query that gives you the count of comments for records where the comment is not empty.
6. Create a CSV file in HDFS with the results of the query from the previous step. The syntax to use is the following:

	```
	INSERT OVERWRITE DIRECTORY '[[hdfs/s3 output directory]]'
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ',' 
	select ...;
	```
	Where:
	- `[[hdfs/s3 output directory]]` is the name of a **directory** to be created, where the results files from the MapReduce process will be placed.
	- `select ...` is the query





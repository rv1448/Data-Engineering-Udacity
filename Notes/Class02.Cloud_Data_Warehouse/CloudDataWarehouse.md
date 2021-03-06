####Implementing Data Warehouses on AWS
` Better Choices Based on Scenario `
Scalability    | Cloud
Operation Cost | On Premise 
Up Front Cost  | Cloud 
Elasticity     | Cloud

` Red Shift Architechture `
- Leader note 
	- Interacts with outside world 
	- Coordinates Compute Nodes 
	- Optimizes query execution
- 1+ Compute note
	- Each with has CPU, Memory and Disk 
	- Scale up: Get more powerful nodes 
	- Scale out: get more nodes 
	
	- Each compute node is divided into logical Slices 
	- A cluster with `n Slices` can process `n Partitions` of a table simultaneously
	- Total number of EC2 instances = number of nodes 
	- Each slice = atleast 1 CPU 
	- Ex: 4 nodes, and each node contain 8 slices and Max parallalism = 32 partitions 

` RedShift & ETL in Context `	

	 -------
        |       |
	|	|  	 
	|	|
	|	|
	 ------

`Ingesting at Scale`
	- To Transfer data from an S3 staging area to redshift use the COPY command 
	- if file is large break up to multiple files 
		- ingest in parallel
		- Common prefix 
		- Manifest file 
	- have Staging area in same AWS region as REDSHIFT 
	- Better to compress all the CSV files 

`Common prefix example`
	``` sql 
	COPY tablename FROM 's3://location/part'
	CREDENTIALS 'aws_iam_role=arn:iam::/dwhrole'
	gzip DELIMITER ';' REGION 'us-west-2';
	```
`Problems with quick lanch cluster`
	- Redshift has to act as user who has READ access to S3 - Impersonation
	- Other BI apps need to connect using TCP ports to pull the data

`IAC - Infracstructure as Code`
	- AWS cli scripts 
	- AWS sdk
		- Create a resources with IAM user that has prevlilages 
	- Amazon cloud formation
``` python 
redshift = boto3.client('redshift',region_name='us-west-2',aws_access_key_id=KEY
			,aws_secret_access_key=SECRET)
response = redshift.create_cluster(
				  ClusterType=DWH_CLUSTER_TYPE,
				  NodeType=NODE_TYPE,
				  NumberOfNodes=int(DWH_NUM_NODES),
				  DBname=DWH_DB,
				  ClusterIdentifier=DWH_CLUSTER_IDENTIFIER,
				  MasterUsername=DWH_DB_USER,
				  MasterUserPassword=DWH_DB_PASSWORD,
				  IamRoles=[roleArn]
				  )
dwhrole = iam.create_role(
			 Path='/'
			 Rolename = DWH_IAM_ROLE_NAME.
			 Description='Allows Redshift clusters to call AWS services on your behalf',
			 AssumeRolePolicyDocument = json.dumps(
			  {'Statement': [{'Action': 'sts.AssumeRole',
			  'Effect': 'Allow',
			  'Principal': {'Service':'redshift.amazon.aws.com'}}],
			  'Version': '2012-10-17'
			  }
			  )
iam.attach_role_policy(Rolename=DWH_IAM_ROLE_NAME,
			PolicyArn:'AmazonS3ReadOnly'
			)['ResponseMetadata']['HTTPStatusCode']
```

`Redshift Table Design`
- Distribution style 
	- EVEN
		- Round Robin will distribute even rows 
		- Each SLice(Partition) will get even rows 
		- Data has to be moved aroud a lot in JOIN 
		- Eg: DIm table distributed and will lead to shuffling
		- Large tables
	- ALL
		- Small table will be copied to each slice 
		- This is when table is less number of rows
	- AUTO
		- Redshift takes care of the distribution style
	- KEY
		- Rows having similar values are placed in same slice
		- usually sk columns are candidates 
		- same FACT sk has to distributed in same way 
		- Skewed dimensions 
		- COLOCATES same rows into the same slice 
- Sorting key 
	- upon loading rows are sorted before distribution to slices 
	- Minimizes the querytime since each node already has contigous range of rows based on the sorting key 
	- Useful for columns that are used frquently in sorting like the date dimension and its corresponding foreign key in the fact table 
	- A column can be dist key and Sort key 
``` sql 
## Load partitioned data into cluster 
copy table from s3://
credentials 'aws_iam_role' gzip delimiter ';' compupdate off region 'us-west-2'
```
` Automate ETL loading into redshift`



`Special Notes`
* ETL from Other Sources 
	- SSH into EC2 machines from REDSHIFT server	

`Follow up`
	- Curated buckets 
	- 


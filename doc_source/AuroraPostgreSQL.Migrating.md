# Migrating data to Amazon Aurora with PostgreSQL compatibility<a name="AuroraPostgreSQL.Migrating"></a>

You have several options for migrating data from your existing database to an Amazon Aurora PostgreSQL\-Compatible Edition DB cluster\. Your migration options also depend on the database that you are migrating from and the size of the data that you are migrating\. Following are your options:

**[ Migrating an RDS for PostgreSQL DB instance using a snapshot](#AuroraPostgreSQL.Migrating.RDSPostgreSQL.Import.Console)**  
You can migrate data directly from an RDS for PostgreSQL DB snapshot to an Aurora PostgreSQL DB cluster\.

**[ Migrating an RDS for PostgreSQL DB instance using an Aurora read replica](#AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica)**  
You can also migrate from an RDS for PostgreSQL DB instance by creating an Aurora PostgreSQL read replica of an RDS for PostgreSQL DB instance\. When the replica lag between the RDS for PostgreSQL DB instance and the Aurora PostgreSQL read replica is zero, you can stop replication\. At this point, you can make the Aurora read replica a standalone Aurora PostgreSQL DB cluster for reading and writing\.

**[Importing S3 data into Aurora PostgreSQL ](USER_PostgreSQL.S3Import.md) **  
You can migrate data by importing it from Amazon S3 into a table belonging to an Aurora PostgreSQL DB cluster\. 

**Migrating from a database that is not PostgreSQL\-compatible**  
You can use AWS Database Migration Service \(AWS DMS\) to migrate data from a database that is not PostgreSQL\-compatible\. For more information on AWS DMS, see [What is AWS Database Migration Service?](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html) in the *AWS Database Migration Service User Guide*\. 

For a list of AWS Regions where Aurora is available, see [Amazon Aurora](https://docs.aws.amazon.com/general/latest/gr/rande.html#aurora) in the *AWS General Reference*\.

**Important**  
If you plan to migrate an RDS for PostgreSQL DB instance to an Aurora PostgreSQL DB cluster in the near future, we strongly recommend that you disable auto minor version upgrades for the DB instance early in the migration planning phase\. Migration to Aurora PostgreSQL might be delayed if the RDS for PostgreSQL version isn't yet supported by Aurora PostgreSQL\. For information about Aurora PostgreSQL versions, see [ Engine versions for Amazon Aurora PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Updates.20180305.html)\.

## Migrating a snapshot of an RDS for PostgreSQL DB instance to an Aurora PostgreSQL DB cluster<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Import.Console"></a>

To create an Aurora PostgreSQL DB cluster, you can migrate a DB snapshot of an RDS for PostgreSQL DB instance\. The new Aurora PostgreSQL DB cluster is populated with the data from the original RDS for PostgreSQL DB instance\. For information about creating a DB snapshot, see [Creating a DB snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateSnapshot.html)\.

In some cases, the DB snapshot might not be in the AWS Region where you want to locate your data\. If so, use the Amazon RDS console to copy the DB snapshot to that AWS Region\. For information about copying a DB snapshot, see [Copying a DB snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CopySnapshot.html)\.

You can migrate RDS for PostgreSQL snapshots that are compatible with the Aurora PostgreSQL versions available in the given AWS Region\. For example, a snapshot from an RDS for PostgreSQL 11\.1 DB instance can be migrated to Aurora PostgreSQL version 11\.4, 11\.7, 11\.8, or 11\.9 in the US West \(N\. California\) Region\. An RDS PostgreSQL 10\.11 snapshot can be migrated to Aurora PostgreSQL 10\.11, 10\.12, 10\.13, and 10\.14\. In other words, the RDS for PostgreSQL snapshot must use the same or a lower minor version as the Aurora PostgreSQL\. 

You can also choose for your new Aurora PostgreSQL DB cluster to be encrypted at rest by using an AWS KMS key\. This option is available only for unencrypted DB snapshots\.

To migrate an RDS for PostgreSQL DB snapshot to an Aurora PostgreSQL DB cluster, you can use the AWS Management Console, the AWS CLI, or the RDS API\. When you use the AWS Management Console, the console takes the actions necessary to create both the DB cluster and the primary instance\.

### Console<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Import.Console.console"></a>

**To migrate a PostgreSQL DB snapshot by using the RDS console**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. Choose **Snapshots**\.

1. On the **Snapshots** page, choose the RDS for PostgreSQL snapshot that you want to migrate into an Aurora PostgreSQL DB cluster\.

1. Choose **Actions** then choose **Migrate snapshot**\.

1. Set the following values on the **Migrate database** page:
   + **DB engine version**: Choose a DB engine version you want to use for the new migrated instance\.
   + **DB instance identifier**: Enter a name for the DB cluster that is unique for your account in the AWS Region that you chose\. This identifier is used in the endpoint addresses for the instances in your DB cluster\. You might choose to add some intelligence to the name, such as including the AWS Region and DB engine that you chose, for example **aurora\-cluster1**\.

     The DB instance identifier has the following constraints:
     + It must contain 1–63 alphanumeric characters or hyphens\.
     + Its first character must be a letter\.
     + It can't end with a hyphen or contain two consecutive hyphens\.
     + It must be unique for all DB instances per AWS account, per AWS Region\.
   + **DB instance class**: Choose a DB instance class that has the required storage and capacity for your database, for example `db.r3.large`\. Aurora cluster volumes automatically grow as the amount of data in your database increases\. 128 tebibytes \(TiB\) So you only need to choose a DB instance class that meets your current storage requirements\. For more information, see [Overview of Aurora storage](Aurora.Overview.StorageReliability.md#Aurora.Overview.Storage)\.
   + **Virtual private cloud \(VPC\)**: If you have an existing VPC, then you can use that VPC with your Aurora PostgreSQL DB cluster by choosing your VPC identifier, for example `vpc-a464d1c1`\. For information on using an existing VPC, see [How to create a VPC for use with Amazon Aurora](Aurora.CreateVPC.md)\.

     Otherwise, you can choose to have Amazon RDS create a VPC for you by choosing **Create new VPC**\. 
   + **Subnet group**: If you have an existing subnet group, then you can use that subnet group with your Aurora PostgreSQL DB cluster by choosing your subnet group identifier, for example `gs-subnet-group1`\.
   + **Public access**: Choose **No** to specify that instances in your DB cluster can only be accessed by resources inside of your VPC\. Choose **Yes** to specify that instances in your DB cluster can be accessed by resources on the public network\.
**Note**  
Your production DB cluster might not need to be in a public subnet, because only your application servers require access to your DB cluster\. If your DB cluster doesn't need to be in a public subnet, set **Public access** to **No**\.
   + **VPC security group**: Choose a VPC security group to allow access to your database\.
   + **Availability Zone**: Choose the Availability Zone to host the primary instance for your Aurora PostgreSQL DB cluster\. To have Amazon RDS choose an Availability Zone for you, choose **No preference**\.
   + **Database port**: Enter the default port to be used when connecting to instances in the Aurora PostgreSQL DB cluster\. The default is `5432`\.
**Note**  
You might be behind a corporate firewall that doesn't allow access to default ports such as the PostgreSQL default port, 5432\. In this case, provide a port value that your corporate firewall allows\. Remember that port value later when you connect to the Aurora PostgreSQL DB cluster\.
   + **Enable Encryption**: Choose **Enable Encryption** for your new Aurora PostgreSQL DB cluster to be encrypted at rest\. Also choose a KMS key as the **AWS KMS key** value\.
   + **Auto minor version upgrade**: Choose **Enable auto minor version upgrade** to enable your Aurora PostgreSQL DB cluster to receive minor PostgreSQL DB engine version upgrades automatically when they become available\.

     The **Auto minor version upgrade** option only applies to upgrades to PostgreSQL minor engine versions for your Aurora PostgreSQL DB cluster\. It doesn't apply to regular patches applied to maintain system stability\.

1. Choose **Migrate** to migrate your DB snapshot\. 

1. Choose **Databases** to see the new DB cluster\. Choose the new DB cluster to monitor the progress of the migration\. On the **Connectivity & security** tab, you can find the cluster endpoint to use for connecting to the primary writer instance of the DB cluster\. For more information on connecting to an Aurora PostgreSQL DB cluster, see [Connecting to an Amazon Aurora DB cluster](Aurora.Connecting.md)\. 

### AWS CLI<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Import.Console.CLI"></a>

Using the AWS CLI to migrate an RDS for PostgreSQL DB snapshot to an Aurora PostgreSQL involves two separate AWS CLI commands\. First, you use the `restore-db-cluster-from-snapshot` AWS CLI command create a new Aurora PostgreSQL DB cluster\. You then use the `create-db-instance` command to create the primary DB instance in the new cluster to complete the migration\. The following procedure creates an Aurora PostgreSQL DB cluster with primary DB instance that has the same configuration as the DB instance used to create the snapshot\. 

**To migrate an RDS for PostgreSQL DB snapshot to an Aurora PostgreSQL DB cluster**

1. Use the [describe\-db\-snapshots](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-snapshots.html) command to obtain information about the DB snapshot you want to migrate\. You can specify either the `--db-instance-identifier` parameter or the `--db-snapshot-identifier` in the command\. If you don't specify one of these parameters, you get all snapshots\. 

   ```
   aws rds describe-db-snapshots --db-instance-identifier <your-db-instance-name>
   ```

1. The command returns all configuration details for any snapshots created from the DB instance specified\. In the output, find the snapshot that you want to migrate and locate its Amazon Resource Name \(ARN\)\. To learn more about Amazon RDS ARNs, see [Amazon Relational Database Service \(Amazon RDS\)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html#arn-syntax-rds)\. An ARN looks similar to the output following\.

   ```
   “DBSnapshotArn": "arn:aws:rds:aws-region:111122223333:snapshot:<snapshot_name>"
   ```

   Also in the output you can find configuration details for the RDS for PostgreSQL DB instance, such as the engine version, allocated storage, whether or not the DB instance is encrypted, and so on\. 

1. Use the [restore\-db\-cluster\-from\-snapshot](https://docs.aws.amazon.com/cli/latest/reference/rds/restore-db-cluster-from-snapshot.html) command to start the migration\. Specify the following parameters:
   + `--db-cluster-identifier` – The name that you want to give to the Aurora PostgreSQL DB cluster\. This Aurora DB cluster is the target for your DB snapshot migration\. 
   + `--snapshot-identifier` – The Amazon Resource Name \(ARN\) of the DB snapshot to migrate\. 
   + `--engine` – Specify `aurora-postgresql` for the Aurora DB cluster engine\.
   + `--kms-key-id` – This optional parameter lets you create an encrypted Aurora PostgreSQL DB cluster from an unencrypted DB snapshot\. It also lets you choose a different encryption key for the DB cluster than the key used for the DB snapshot\.
**Note**  
You can't create an unencrypted Aurora PostgreSQL DB cluster from an encrypted DB snapshot\. 

   Without the `--kms-key-id` parameter specified as shown following, the [restore\-db\-cluster\-from\-snapshot](https://docs.aws.amazon.com/cli/latest/reference/rds/restore-db-cluster-from-snapshot.html) AWS CLI command creates an empty Aurora PostgreSQL DB cluster that's either encrypted using the same key as the DB snapshot or is unencrypted if the source DB snapshot isn't encrypted\. 

   For Linux, macOS, or Unix:

   ```
   aws rds restore-db-cluster-from-snapshot \
       --db-cluster-identifier cluster-name \
       --snapshot-identifier arn:aws:rds:aws-region:111122223333:snapshot:your-snapshot-name \
       --engine aurora-postgresql
   ```

   For Windows:

   ```
   aws rds restore-db-cluster-from-snapshot ^
       --db-cluster-identifier new_cluster ^
       --snapshot-identifier arn:aws:rds:aws-region:111122223333:snapshot:your-snapshot-name ^
       --engine aurora-postgresql
   ```

1. The command returns details about the Aurora PostgreSQL DB cluster that's being created for the migration\. You can check the status of the Aurora PostgreSQL DB cluster by using the [describe\-db\-clusters](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-clussters.html) AWS CLI command\.

   ```
   aws rds describe-db-clusters --db-cluster-idenfifier cluster-name
   ```

1. When the DB cluster becomes "available", you use [create\-db\-instance](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance.html) command to populate the Aurora PostgreSQL DB cluster with the DB instance based on your Amazon RDS DB snapshot\. Specify the following parameters:
   + `--db-cluster-identifier` – The name of the new Aurora PostgreSQL DB cluster that you created in the previous step\. 
   + `--db-instance-identifier` – The name you want to give to the DB instance\. This instance becomes the primary node in your Aurora PostgreSQL DB cluster\.
   + `----db-instance-class ` – Specify the DB instance class to use\. Choose from among the DB instance classes supported by the Aurora PostgreSQL version to which you're migrating\. For more information, see [DB instance class types](Concepts.DBInstanceClass.md#Concepts.DBInstanceClass.Types) and [Supported DB engines for DB instance classes](Concepts.DBInstanceClass.md#Concepts.DBInstanceClass.SupportAurora)\.
   + `--engine` – Specify `aurora-postgresql` for the DB instance\.

   You can also create the DB instance with a different configuration than the source DB snapshot, by passing in the appropriate options in the `create-db-instance` AWS CLI command\. For more information, see the [create\-db\-instance](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance.html) command\. 

   For Linux, macOS, or Unix:

   ```
   aws rds create-db-instance \
       --db-cluster-identifier cluster-name \
       --db-instance-identifier --db-instance-class db.instance.class \
       --engine aurora-postgresql
   ```

   For Windows:

   ```
   aws rds create-db-instance ^
       --db-cluster-identifier cluster-name ^
       --db-instance-identifier --db-instance-class db.instance.class ^
       --engine aurora-postgresql
   ```

When the migration process completes, the Aurora PostgreSQL cluster has a populated primary DB instance\. 

## Migrating data from an RDS for PostgreSQL DB instance to an Aurora PostgreSQL DB cluster using an Aurora read replica<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica"></a>

You can migrate from an RDS for PostgreSQL DB instance to an Aurora PostgreSQL DB cluster by using an Aurora read replica\. When you need to migrate from an RDS for PostgreSQL DB instance to an Aurora PostgreSQL DB cluster, we recommend using this approach\. 

In this case, Amazon RDS uses the PostgreSQL DB engine's streaming replication functionality to create a special type of DB cluster for the source PostgreSQL DB instance\. This type of DB cluster is called an Aurora read replica\. Updates made to the source RDS for PostgreSQL DB instance are asynchronously replicated to the Aurora read replica\. 

**Topics**
+ [Overview of migrating data by using an Aurora read replica](#AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Intro)
+ [Preparing to migrate data by using an Aurora read replica](#AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Prepare)
+ [Creating an Aurora read replica](#AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Create)
+ [Promoting an Aurora read replica](#AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Promote)

### Overview of migrating data by using an Aurora read replica<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Intro"></a>

To migrate from an RDS for PostgreSQL DB instance to an Aurora PostgreSQL DB cluster, we recommend creating an Aurora read replica of your source RDS for PostgreSQL DB instance\. When the replica lag between the RDS for PostgreSQL DB instance and the Aurora PostgreSQL Read Replica is zero, you can stop replication\. At this point, you can promote the Aurora read replica to be a standalone Aurora PostgreSQL DB cluster\. This standalone DB cluster can then accept write loads\. 

Be prepared for migration to take a while, roughly several hours per tebibyte \(TiB\) of data\. While the migration is in progress, your RDS for PostgreSQL instance accumulates write ahead log \(WAL\) segments\. Make sure that your Amazon RDS instance has sufficient storage capacity for these segments\. 

When you create an Aurora read replica of an RDS for PostgreSQL DB instance, Amazon RDS creates a DB snapshot of your source RDS for PostgreSQL DB instance\. This snapshot is private to Amazon RDS and incurs no charges\. Amazon RDS then migrates the data from the DB snapshot to the Aurora read replica\. After the DB snapshot data is migrated to the new Aurora PostgreSQL DB cluster, RDS starts replication between your RDS for PostgreSQL DB instance and the Aurora PostgreSQL DB cluster\. 

You can only have one Aurora read replica for an RDS for PostgreSQL DB instance\. If you try to create an Aurora read replica for your RDS for PostgreSQL instance and you already have an Aurora read replica or a cross\-region read replica, the request is rejected\. 

**Note**  
Replication issues can arise due to feature differences between Aurora PostgreSQL and the PostgreSQL engine version of your RDS for PostgreSQL DB instance that is the replication source\. You can replicate only from an RDS for PostgreSQL instance that is compatible with the Aurora PostgreSQL version\. The RDS for PostgreSQL version must be lower than or equal to a supported Aurora PostgreSQL version in the same major version\. For example, you can replicate data between an RDS for PostgreSQL version 11\.7 DB instance and an Aurora PostgreSQL version 11\.7 or higher 11 version DB cluster, but not an Aurora PostgreSQL version 11\.6 DB cluster\. For information about Aurora PostgreSQL versions, see [ Aurora PostgreSQL releases and engine versions](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Updates.20180305.html) in the *Aurora User Guide\.* If you encounter an error, you can find help in the [Amazon RDS community forum](https://forums.aws.amazon.com/forum.jspa?forumID=60) or by contacting AWS Support\.

For more information on PostgreSQL read replicas, see [Working with read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html) in the *Amazon RDS User Guide*\.

### Preparing to migrate data by using an Aurora read replica<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Prepare"></a>

Before you migrate data from your RDS for PostgreSQL instance to an Aurora PostgreSQL cluster, make sure that your instance has sufficient storage capacity\. This storage capacity is for the write ahead log \(WAL\) segments that accumulate during the migration\. There are several metrics to check for this, described following\. 


| Metric | Description | 
| --- | --- | 
|  `FreeStorageSpace`  |  The available storage space\. Units: Bytes  | 
|  `OldestReplicationSlotLag`  |  The size of the lag for WAL data in the replica that is lagging the most\. Units: Megabytes  | 
|  `RDSToAuroraPostgreSQLReplicaLag`  |  The amount of time in seconds that an Aurora PostgreSQL DB cluster lags behind the source RDS DB instance\.  | 
|  `TransactionLogsDiskUsage`  |  The disk space used by the transaction logs\. Units: Megabytes  | 

For more information about monitoring your RDS instance, see [Monitoring](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Monitoring.html) in the *Amazon RDS User Guide*\.

### Creating an Aurora read replica<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Create"></a>

You can create an Aurora read replica for an RDS for PostgreSQL DB instance by using the console or the AWS CLI\.

#### Console<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Create.Console"></a>

**To create an Aurora read replica from a source PostgreSQL DB instance**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Databases**\. 

1. Choose the RDS for PostgreSQL DB instance that you want to use as the source for your Aurora read replica\. For **Actions**, choose **Create Aurora read replica**\.  
![\[Create Aurora read replica\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/Aurorapgres-migrate.png)

1. Choose the DB cluster specifications that you want to use for the Aurora read replica, as described in the following table\.     
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Migrating.html)

1. Choose **Create read replica**\.

#### AWS CLI<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Create.CLI"></a>

To create an Aurora read replica from a source RDS for PostgreSQL DB instance, use the [https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-cluster.html](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-cluster.html) and [https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance.html](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance.html) AWS CLI commands to create a new Aurora PostgreSQL DB cluster\. When you call the create\-db\-cluster command, include the `--replication-source-identifier` parameter to identify the Amazon Resource Name \(ARN\) for the source RDS for PostgreSQL DB instance\. For more information about Amazon RDS ARNs, see [Amazon Relational Database Service \(Amazon RDS\)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html#arn-syntax-rds) in the *AWS General Reference*\. 

Don't specify the master user name, master password, or database name\. The Aurora read replica uses the same master user name, master password, and database name as the source RDS for PostgreSQL DB instance\. 

For Linux, macOS, or Unix:

```
aws rds create-db-cluster --db-cluster-identifier sample-replica-cluster --engine aurora-postgresql \
    --db-subnet-group-name mysubnetgroup --vpc-security-group-ids sg-c7e5b0d2 \
    --replication-source-identifier arn:aws:rds:us-west-2:123456789012:db:master-postgresql-instance
```

For Windows:

```
aws rds create-db-cluster --db-cluster-identifier sample-replica-cluster --engine aurora-postgresql ^
    --db-subnet-group-name mysubnetgroup --vpc-security-group-ids sg-c7e5b0d2 ^
    --replication-source-identifier arn:aws:rds:us-west-2:123456789012:db:master-postgresql-instance
```

If you use the console to create an Aurora read replica, then RDS automatically creates the primary instance for your DB cluster Aurora Read Replica\. If you use the CLI to create an Aurora read replica, you must explicitly create the primary instance for your DB cluster\. The primary instance is the first instance that is created in a DB cluster\.

You can create a primary instance for your DB cluster by using the [https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance.html](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance.html) CLI command with the following parameters:
+ `--db-cluster-identifier`

  The name of your DB cluster\.
+ `--db-instance-class`

  The name of the DB instance class to use for your primary instance\.
+ `--db-instance-identifier`

  The name of your primary instance\.
+ `--engine aurora-postgresql`

  The database engine to use\.

In the following example, you create a primary instance named *`myreadreplicainstance`* for the DB cluster named *`myreadreplicacluster`*\. You do this using the DB instance class specified in *`myinstanceclass`*\.

**Example**  
For Linux, macOS, or Unix:  

```
aws rds create-db-instance \
    --db-cluster-identifier myreadreplicacluster \
    --db-instance-class myinstanceclass \
    --db-instance-identifier myreadreplicainstance \
    --engine aurora-postgresql
```
For Windows:  

```
aws rds create-db-instance ^
    --db-cluster-identifier myreadreplicacluster ^
    --db-instance-class myinstanceclass ^
    --db-instance-identifier myreadreplicainstance ^
    --engine aurora-postgresql
```

#### RDS API<a name="Aurora.Migration.RDSPostgreSQL.Create.API"></a>

To create an Aurora read replica from a source RDS for PostgreSQL DB instance, use the RDS API operations [https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBCluster.html](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBCluster.html) and [https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html) to create a new Aurora DB cluster and primary instance\. Don't specify the master user name, master password, or database name\. The Aurora read replica uses the same master user name, master password, and database name as the source RDS for PostgreSQL DB instance\. 

You can create a new Aurora DB cluster for an Aurora read replica from a source RDS for PostgreSQL DB instance\. To do so, use the RDS API operation [https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBCluster.html](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBCluster.html) with the following parameters:
+ `DBClusterIdentifier`

  The name of the DB cluster to create\.
+ `DBSubnetGroupName`

  The name of the DB subnet group to associate with this DB cluster\.
+ `Engine=aurora-postgresql`

  The name of the engine to use\.
+ `ReplicationSourceIdentifier`

  The Amazon Resource Name \(ARN\) for the source PostgreSQL DB instance\. For more information about Amazon RDS ARNs, see [Amazon Relational Database Service \(Amazon RDS\)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html#arn-syntax-rds) in the *Amazon Web Services General Reference\.*
+ `VpcSecurityGroupIds`

  The list of Amazon EC2 VPC security groups to associate with this DB cluster\.

See an example with the RDS API operation [https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBCluster.html](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBCluster.html)\.

If you use the console to create an Aurora read replica, then Amazon RDS automatically creates the primary instance for your DB cluster Aurora Read Replica\. If you use the CLI to create an Aurora read replica, you must explicitly create the primary instance for your DB cluster\. The primary instance is the first instance that is created in a DB cluster\.

You can create a primary instance for your DB cluster by using the RDS API operation [https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html) with the following parameters:
+ `DBClusterIdentifier`

  The name of your DB cluster\.
+ `DBInstanceClass`

  The name of the DB instance class to use for your primary instance\.
+ `DBInstanceIdentifier`

  The name of your primary instance\.
+ `Engine=aurora-postgresql`

  The name of the engine to use\.

See an example with the RDS API operation [https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html)\.

### Promoting an Aurora read replica<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Promote"></a>

After migration completes, you can promote the Aurora read replica to a standalone DB cluster\. You then direct your client applications to the endpoint for the Aurora read replica\. For more information on the Aurora endpoints, see [Amazon Aurora connection management](Aurora.Overview.Endpoints.md)\. Promotion should complete fairly quickly\. You can't delete the primary PostgreSQL DB instance or unlink the DB instance and the Aurora read replica until the promotion is complete\.

Before you promote your Aurora read replica, stop any transactions from being written to the source RDS for PostgreSQL DB instance\. Then wait for the replica lag on the Aurora read replica to reach zero\. For more information, see [Monitoring Aurora PostgreSQL replication](AuroraPostgreSQL.Replication.md#AuroraPostgreSQL.Replication.Monitoring) and [Monitoring Amazon Aurora metrics with Amazon CloudWatch](Aurora.Monitoring.md)\.

#### Console<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Promote.Console"></a>

**To promote an Aurora read replica to an Aurora DB cluster**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Databases**\. 

1. Choose the DB instance for the Aurora read replica\. 

1. For **Actions**, choose **Promote**\. 

1. Choose **Promote read replica**\.

#### AWS CLI<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Promote.CLI"></a>

To promote an Aurora read replica to a stand\-alone DB cluster, use the [https://docs.aws.amazon.com/cli/latest/reference/rds/promote-read-replica-db-cluster.html](https://docs.aws.amazon.com/cli/latest/reference/rds/promote-read-replica-db-cluster.html) AWS CLI command\. 

**Example**  
For Linux, macOS, or Unix:  

```
aws rds promote-read-replica-db-cluster \
    --db-cluster-identifier myreadreplicacluster
```
For Windows:  

```
aws rds promote-read-replica-db-cluster ^
    --db-cluster-identifier myreadreplicacluster
```

#### RDS API<a name="AuroraPostgreSQL.Migrating.RDSPostgreSQL.Replica.Promote.API"></a>

To promote an Aurora read replica to a stand\-alone DB cluster, use the RDS API operation [PromoteReadReplicaDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_PromoteReadReplicaDBCluster.html)\. 

After you promote your read replica, confirm that the promotion has completed by using the following procedure\.

**To confirm that the Aurora read replica was promoted**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Events**\.

1. On the **Events** page, verify that there is an event, `Promoted read replica cluster to a stand-alone database cluster` for the cluster that you promoted\.

After promotion is complete, the primary RDS for PostgreSQL DB instance and the Aurora Read Replica are unlinked\. At this point, you can safely delete the DB instance if you want to\.
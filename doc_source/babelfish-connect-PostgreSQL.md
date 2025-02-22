# Using a PostgreSQL client to connect to your DB cluster<a name="babelfish-connect-PostgreSQL"></a>

You can use a PostgreSQL client to connect to Babelfish on the PostgreSQL port\. 

## Using psql to connect to the DB cluster<a name="babelfish-connect-psql"></a>

You can query an Aurora PostgreSQL DB cluster that supports Babelfish with the `psql` command line client\. When connecting, use the PostgreSQL port\. Use the following command to connect to Babelfish with the psql client:

```
psql "host=babelfish_db.cluster-123456789012
	port=portNumber dbname=babelfish_db user=userName"
```

The parameters are as follows:
+ `host` – The host name of the DB cluster \(cluster endpoint\) that you want to access
+ `port` – The PostgreSQL port number used to connect to your DB instance
+ `dbname` – `babelfish_db`
+ `user` – The database user account that you want to access
+ `password` – The password of the database user

When you run a SQL command on the psql client, you end the command with a semicolon\. For example, the following SQL command queries the [pg\_tables system view](https://www.postgresql.org/docs/current/view-pg-tables.html) to return information about each table in the database\.

`SELECT * FROM pg_tables;`

The psql client also has a set of built\-in metacommands\. A *metacommand* is a shortcut that adjusts formatting or provides a shortcut that returns meta\-data in an easy\-to\-use format\. For example, the following metacommand returns similar information to the previous SQL command:

`\d`

Metacommands don't need to be terminated with a semicolon \(;\)\.

To exit the psql client, enter `\q`\.

For more information about using the psql client to query an Aurora PostgreSQL cluster, see [the PostgreSQL documentation](https://www.postgresql.org/docs/14/app-psql.html)\.

## Using pgAdmin to connect to the DB cluster<a name="babelfish-connect-pgadmin"></a>

You can use the pgAdmin client to access your data in native PostgreSQL dialect\. 

**To connect to the cluster with the pgAdmin client**

1. Download and install the pgAdmin client from the [pgAdmin website](https://www.pgadmin.org/)\.

1. Open the client and authenticate with pgAdmin\.

1. Open the context \(right\-click\) menu for **Servers**, and then choose **Create**, **Server**\.  
![\[\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/pgAdmin1.png)

1. Enter information in the **Create \- Server** dialog box\. 

   On the **Connection** tab, add the Aurora PostgreSQL cluster address for **Host** and the PostgreSQL port number \(by default, 5432\) for **Port**\. Provide authentication details, and choose **Save**\.  
![\[\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/pgAdmin2.png)

After connecting, you can use pgAdmin functionality to monitor and manage your Aurora PostgreSQL cluster on the PostgreSQL port\.

![\[\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/pgAdmin3.png)

For more details about using pgAdmin, visit the [pgAdmin web site](https://www.pgadmin.org/)\.
# Monitoring an Amazon Aurora DB cluster<a name="MonitoringAurora"></a>

Amazon Aurora uses a cluster of replicated database servers\. Typically, monitoring an Aurora cluster requires checking the health of multiple DB instances\. The instances might have specialized roles, handling mostly write operations, only read operations, or a combination\. You also monitor the overall health of the cluster by measuring the *replication lag*\. This is the amount of time for changes made by one DB instance to be available to the other instances\. 

**Topics**
+ [Overview of monitoring Amazon Aurora](MonitoringOverview.md)
+ [Viewing key monitoring information](accessing-monitoring.md)
+ [Monitoring Amazon Aurora metrics with Amazon CloudWatch](Aurora.Monitoring.md)
+ [Monitoring DB load with Performance Insights on Amazon Aurora](USER_PerfInsights.md)
+ [Analyzing performance anomalies with DevOps Guru for RDS](devops-guru-for-rds.md)
+ [Monitoring the OS by using Enhanced Monitoring](USER_Monitoring.OS.md)
+ [Working with Amazon RDS events](working-with-events.md)
+ [Working with Amazon Aurora database log files](USER_LogAccess.md)
+ [Working with AWS CloudTrail and Amazon RDS](logging-using-cloudtrail.md)
+ [Monitoring Amazon Aurora using Database Activity Streams](DBActivityStreams.md)
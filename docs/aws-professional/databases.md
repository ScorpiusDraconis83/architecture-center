---
title: Compare AWS and Azure database technology
description: Compare database technology differences between Azure and AWS. Review the Amazon RDS and Azure relational database services. See equivalents for analytics and big data.
author: splitfinity-zz-zz
ms.author: yubaijna
ms.date: 07/25/2022
ms.topic: conceptual
ms.subservice: cloud-fundamentals
ms.collection: 
 - migration
 - aws-to-azure
 - gcp-to-azure
---

# Relational database technologies on Azure and AWS

## Amazon RDS and Azure relational database services

Azure provides several different relational database services that are the equivalent of AWS' Relational Database Service (Amazon RDS). These include:

- [SQL Database](/azure/sql-database/sql-database-technical-overview)
- [Azure Database for MySQL](/azure/mysql/overview)
- [Azure Database for PostgreSQL](/azure/postgresql/overview)

Other database engines such as [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server), [Oracle](https://azure.microsoft.com/campaigns/oracle), and [MySQL](/azure/mysql) can be deployed using Azure VM Instances.

Costs for Amazon RDS are determined by the amount of hardware resources that your instance uses, like CPU, RAM, storage, and network bandwidth. In the Azure database services, cost depends on your database size, concurrent connections, and throughput levels.

### See also

- [Azure SQL Database Tutorials](/azure/azure-sql/database/single-database-create-quickstart)

- [Azure SQL Managed Instance](/azure/azure-sql/managed-instance/sql-managed-instance-paas-overview)

- [Configure geo-replication for Azure SQL Database with the Azure portal](/azure/azure-sql/database/active-geo-replication-configure-portal)

- [Introduction to Azure Cosmos DB: A NoSQL JSON Database](/azure/cosmos-db/sql-api-introduction)

- [How to use Azure Table storage from Node.js](/azure/cosmos-db/table-storage-how-to-use-nodejs)

## Analytics and big data

Azure provides a package of products and services designed to capture, organize, analyze, and visualize large amounts of data consisting of the following services:

- [HDInsight](/azure/hdinsight): managed Apache distribution that includes Hadoop, Spark, Storm, or HBase.

- [Data Factory](/azure/data-factory): provides data orchestration and data pipeline functionality.

- [Azure Synapse Analytics](/azure/synapse-analytics/overview-what-is): an enterprise analytics service that accelerates time to insight, across data warehouses and big data systems.

- [Azure Databricks](/azure/databricks/): a unified analytics platform for data analysts, data engineers, data scientists, and machine learning engineers.

- [Data Lake Store](/azure/data-lake-store): analytics service that brings together enterprise data warehousing and big data analytics. Query data on your terms, using either serverless or dedicated resources—at scale.

- [Machine Learning](/azure/machine-learning): used to build and apply predictive analytics on data.

- [Stream Analytics](/azure/stream-analytics): real-time data analysis.

- [Data Lake Analytics](/azure/data-lake-analytics/data-lake-analytics-overview): large-scale analytics service optimized to work with Data Lake Store

- [Power BI](https://powerbi.microsoft.com): a business analytics service that provides the capabilities to create rich interactive data visualizations.

## Service comparison

| Type | AWS Service | Azure Service | Description |
| -----| ----------- | ------------- | ----------- |
| Relational database | [RDS](https://aws.amazon.com/rds) | [SQL Database](https://azure.microsoft.com/services/sql-database)<br/><br/>[Database for MySQL](https://azure.microsoft.com/services/mysql)<br/><br/>[Database for PostgreSQL](https://azure.microsoft.com/services/postgresql) | Managed relational database services in which resiliency, scale and maintenance are primarily handled by the Azure platform. |
| Serverless relational database | [Amazon Aurora Serverless](https://aws.amazon.com/rds/aurora/serverless) | [Azure SQL Database serverless](/azure/azure-sql/database/serverless-tier-overview)<br/><br/>[Serverless SQL pool in Azure Synapse Analytics](/azure/synapse-analytics/sql/on-demand-workspace-overview) | Database offerings that automatically scales compute based on the workload demand. You're billed per second for the actual compute used (Azure SQL)/data that's processed by your queries (Azure Synapse Analytics Serverless). |
| NoSQL | [DynamoDB](https://aws.amazon.com/dynamodb) (Key-Value)<br/><br/>[SimpleDB](https://aws.amazon.com/simpledb/)<br/><br/>[Amazon DocumentDB](https://aws.amazon.com/documentdb) (Document)<br /><br />[Amazon Neptune](https://aws.amazon.com/neptune/) (Graph) | [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db) | Azure Cosmos DB is a globally distributed, multi-model database that natively supports multiple data models including key-value pairs, documents, graphs, and columnar. |
| Caching | [ElastiCache](https://aws.amazon.com/elasticache)<br /><br />[Amazon MemoryDB for Redis](https://aws.amazon.com/memorydb/) | [Cache for Redis](https://azure.microsoft.com/services/cache) | An in-memory–based, distributed caching service that provides a high-performance store that's typically used to offload nontransactional work from a database. |
| Database migration | [Database Migration Service](https://aws.amazon.com/dms) | [Database Migration Service](https://azure.microsoft.com/campaigns/database-migration) | A service that executes the migration of database schema and data from one database format to a specific database technology in the cloud. |

### See also

- [Cloud-scale analytics](https://azure.microsoft.com/solutions/big-data/#overview)

- [Big data architecture style](../guide/architecture-styles/big-data.yml)

- [Azure Data Lake and Azure HDInsight blog](/archive/blogs/azuredatalake)

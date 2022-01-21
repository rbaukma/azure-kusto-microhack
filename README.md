# Azure Data Explorer Microhack

Kusto Product Group and Microsoft Global Black Belt team are pleased to present this challenge based, collaboration driven, discover-by-doing learning experience to you. Microhacks are divided into two parts to cater enough time for the participants to understand the key concepts of Azure Data Explorer effectively.
- MicroHack 1: Cluster creation and data ingestion
This MicroHack will focus on enabling the participants to design ADX based big data analytics solution, create an ADX cluster, and ingest data into the cluster.
- MicroHack 2: Data exploration and visualization using Kusto Query Language (KQL)
This MicroHack will focus on enabling the participants to write kusto queries to explore and analyse the data stored in the clusters. Participants will also create cool visualizations. It is recommended to complete the MicroHack 1 to begin with this MicroHack.


## MicroHack 1: ADX Cluster Creation and Data Ingestion

### Scenario 

Contoso is a supply chain logistics company that runs a fleet of ships, trucks, and cargo planes to transport and deliver goods around the world. Some of the world’s largest enterprises rely on Contoso’s logistics capabilities to deliver goods to their end customers. Contoso has invested in connecting its fleet with sensors that measure temperature, pressure, humidity, tilt, shock, and light exposure inside its fleet. These sensors emit telemetry data every 1 minute, property data whenever there is a change in the device property, and command data whenever a new command is executed. 

Contoso is looking for a suitable data storage and analytical solution that provides out of the box integration with Azure IoT services such as IoT Hub, Event Hubs as well as can read data from storage accounts. Contoso is developing a SaaS application that will allow its customers to track, trace and monitor their shipments. Contoso wants to offer out of the box visualizations with interactive capabilities to enable its customers to drill-in/drill-out of the data. Contoso will offer its customers to view and analyze last 6 months data. Contoso will retain every customer’s data for up to 1 year. Contoso wants to offer blazing fast loading of visualizations to its customers.
This MicroHack walks through the steps in designing, creating, and configuring Azure Data Explorer clusters keeping in mind these requirements. Once the cluster is deployed, this MicroHack enlists the steps to ingest data into ADX databases and tables using various integration methods such as One Click ingestion.

### Pre-requisites
- An Azure subscription
- (Not applicable for Proctor led events) Deploy IoT Central application, create simulated devices and create Data Exports to Event Hubs and Storage Accounts (use this guide to create this infrastructure). For proctor led events, this infrastructure has been pre-created for you. Proctor will provide connection strings, or SAS tokens at an appropriate stage of the hack.
- Authorization to create an Azure Data Explorer cluster or Synapse Data Explorer Pool

### Overview

Following architecture has been deployed for you, except the ADX cluster and its integration with other Azure services (indicated by dashed lines).
IoT Central acts as the source of telemetry generated by Contoso’s sensors installed on its fleet of trucks, vessels, and airplanes. Telemetry data is streamed on continuous basis to the Event Hub. Device logs, device property changes and commands executed on the devices are stored in a Storage Account as blobs. 

#### Challenge 0: What is Azure Data Explorer and when is it a good fit?

Azure Data Explorer is a fully managed, high-performance, big data analytics platform that makes it easy to analyze high volumes of data in near real time. The Azure Data Explorer toolbox gives you an end-to-end solution for data ingestion, query, visualization, and management.

By analyzing structured, semi-structured, and unstructured data across time series, and by using Machine Learning, Azure Data Explorer makes it simple to extract key insights, spot patterns and trends, and create forecasting models. Azure Data Explorer is scalable, secure, robust, and enterprise-ready, and is useful for log analytics, time series analytics, IoT, and general-purpose exploratory analytics.

Azure Data Explorer capabilities are extended by other services built on its powerful query language, including [Azure Monitor logs](https://docs.microsoft.com/en-us/azure/log-analytics/), [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/), [Time Series Insights](https://docs.microsoft.com/en-us/azure/time-series-insights/), and [Microsoft Defender for Endpoint](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/microsoft-defender-endpoint)

Generally speaking, when you interact with Azure Data Explorer, you're going to go through the following workflow (ADX Microhacks will cover all these steps):
1. Create an ADX cluster - To use Azure Data Explorer you first create a cluster. An Azure Data Explorer cluster is the most basic unit.
2. Create database: Each cluster has one or more databases in that cluster. Each Azure Data Explorer cluster can hold up to 10,000 databases and each database up to 10,000 tables. 
3. Ingest data: Load data into database tables so that you can run queries against it. Azure Data Explorer supports several ingestion methods.
4. Query database: Azure Data Explorer uses the Kusto Query Language, which is an expressive, intuitive, and highly productive query language. It offers a smooth transition from simple one-liners to complex data processing scripts, and supports querying structured, semi-structured, and unstructured (text search) data. 
5. Use the web application to run, review, and share queries and results. You can also send queries programmatically (using an SDK) or to a REST API endpoint. 
6. Visualize results: Use different visual displays of your data in the native Azure Data Explorer Dashboards. You can also display your results using connectors to some of the leading visualization services, such as Power BI and Grafana. 

#### Challenge 1: Create ADX cluster
To use Azure Data Explorer (ADX), you first have to create an ADX cluster, and create one or more databases in that cluster. Each database has tables. Then you can ingest data into a database so that you can run queries against it.

In this challenge, you will design an ADX based architecture, create an ADX cluster and database.
In addition, you will get familiarized with two tools that enables you to connect to your Azure Data Explorer and run queries.

##### Task 1: Create an ADX cluster resource
Sign in to the Azure portal, select the + Create a resource button in the upper-left corner of the portal’s main page.

![Screen capture 1](/assets/images/Challenge1-Task1-Pic1.png)
  
- Search for Azure Data Explorer. Under Azure Data Explorer, select Create.

![Screen capture 1](/assets/images/Challenge1-Task1-Pic2.png)
![Screen capture 1](/assets/images/Challenge1-Task1-Pic3.png)

- Fill out the basic cluster details with the following information.

![Screen capture 1](/assets/images/Challenge1-Task1-Pic4.png)

- Subscription: Use your own subscription
- Resource Group: It's recommended to create a new resource group for the microhack's resources. Call it: <youralias>-microhack-RG
- Cluster name: Must be unique per participants. Call it: <youralias>microhackadx (cluster name must begin with a letter and contain lowercase alphanumeric characters.)
- Region: France Central
-	Enable performance update (EngineV3): keep the default (enabled)
-	Compute specification: For a production system, select the specification that best meets your needs (storage optimized or compute optimized). For this Microhack we can use ‘Compute optimized’, Extra small (2 cores), Standard_D11_v2.
  With various compute SKU options to choose from, you can optimize costs for the performance and hot-cache requirements for your scenario. If you need the most optimal performance for a high query volume, the ideal SKU should be compute-optimized. If you need to query large volumes of data with relatively lower query load, the storage-optimized SKU can help reduce costs and still provide excellent performance. You can read more about ADX’s SKU types [here](https://docs.microsoft.com/en-us/azure/data-explorer/manage-cluster-choose-sku)
- Availability zones: keep the default.
Move to the next tab (“Scale”). Choose how to scale your resource. Select the “Optimized autoscale” option. It’s always recommended to use this option. Optimized Autoscale is a built-in feature that helps clusters perform their best when demand changes. Optimized Autoscale enables your cluster to be performant and cost effective by adding and removing instances based on demand. For this microhack keep the default values (Minimum instance count == 2, Maximum instance count == 3)

You can keep all the other configurations with the default values. 
Select Review + create to review your cluster details. Then, select Create to provision the cluster. Provisioning typically takes about 10 minutes.
Creating an ADX cluster takes in average 10-15 minutes.

When the deployment is complete, select Go to resource. You will be redirected the ADX cluster resource page. On the top of Overview page, you can see the basic details of the cluster, like: the Subscription, the state (running) and the URI.

##### Task 2: Create a Database
- You're now ready for the second step in the process: database creation.
- On the Overview tab, select Create database. Alternatively, you can go to the “Databases” blade.

  ![Screen capture 1](/assets/images/Challenge1-Task1-Pic4.png)

- Fill out the form with the following information.
  
  ![Screen capture 1](/assets/images/Challenge1-Task1-Pic4.png)
  
  | Setting       | Suggested Value   | Field Description                                                             |
  | ------------- | ----------------- | ----------------------------------------------------------------------------- |
  | Admin         | Default selected  | The admin field is disabled. New admins can be added after database creation. |
  | Database Name | TelemetryDatabase | The database name must be unique within the cluster.                          |
  | Retention period | 365	| The time span (in days) for which it's guaranteed that the data is kept available to query. The time span is measured from the time that data is ingested. This is the longer-term storage (in reliable storage) retention. |
  | Cache period	| 31	| The time span (in days) for which to keep frequently queried data available in SSD storage or RAM of the cluster’s VM, rather than in longer-term storage. Azure Data Explorer stores all its ingested data in reliable storage (most commonly Azure Blob Storage), away from its actual processing (such as Azure Compute) nodes. To speed up queries on that data, Azure Data Explorer caches it, or parts of it, on its processing nodes, SSD, or even in RAM. The best query performance is achieved when all ingested data is cached. Sometimes, certain data doesn't justify the cost of keeping it "warm" in local SSD storage. For example, many teams consider that rarely accessed older log records are of lesser importance. They prefer to have reduced performance when querying this data, rather than pay to keep it warm all the time.By increasing the cache policy, more VMs will be required to store data on their SSD/RAM. For Azure Data Explorer cluster, compute cost (VMs) is the most significant part of cluster cost as compared to storage and networking. |
 
  
#### Challenge 2: Create integration with Azure services

#### Challenge 3: Ingest and transform data

#### Challenge 4: Retrieve stats on the ingested data

#### Challenge 5: Setup basic monitoring of the cluster

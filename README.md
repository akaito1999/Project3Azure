# TechConf Registration Website

## Project Overview
The TechConf website allows attendees to register for an upcoming conference. Administrators can also view the list of attendees and notify all attendees via a personalized email message.

The application is currently working but the following pain points have triggered the need for migration to Azure:
 - The web application is not scalable to handle user load at peak
 - When the admin sends out notifications, it's currently taking a long time because it's looping through all attendees, resulting in some HTTP timeout exceptions
 - The current architecture is not cost-effective 

In this project, you are tasked to do the following:
- Migrate and deploy the pre-existing web app to an Azure App Service
- Migrate a PostgreSQL database backup to an Azure Postgres database instance
- Refactor the notification logic to an Azure Function via a service bus queue message

## Dependencies

You will need to install the following locally:
- [Postgres](https://www.postgresql.org/download/)
- [Visual Studio Code](https://code.visualstudio.com/download)
- [Azure Function tools V3](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#install-the-azure-functions-core-tools)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Tools for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack)

## Project Instructions

### Part 1: Create Azure Resources and Deploy Web App
1. Create a Resource group
2. Create an Azure Postgres Database single server
   - Add a new database `techconfdb`
   - Allow all IPs to connect to database server
   - Restore the database with the backup located in the data folder
3. Create a Service Bus resource with a `notificationqueue` that will be used to communicate between the web and the function
   - Open the web folder and update the following in the `config.py` file
      - `POSTGRES_URL`
      - `POSTGRES_USER`
      - `POSTGRES_PW`
      - `POSTGRES_DB`
      - `SERVICE_BUS_CONNECTION_STRING`
4. Create App Service plan
5. Create a storage account
6. Deploy the web app

### Part 2: Create and Publish Azure Function
1. Create an Azure Function in the `function` folder that is triggered by the service bus queue created in Part 1.

      **Note**: Skeleton code has been provided in the **README** file located in the `function` folder. You will need to copy/paste this code into the `__init.py__` file in the `function` folder.
      - The Azure Function should do the following:
         - Process the message which is the `notification_id`
         - Query the database using `psycopg2` library for the given notification to retrieve the subject and message
         - Query the database to retrieve a list of attendees (**email** and **first name**)
         - Loop through each attendee and send a personalized subject message
         - After the notification, update the notification status with the total number of attendees notified
2. Publish the Azure Function

### Part 3: Refactor `routes.py`
1. Refactor the post logic in `web/app/routes.py -> notification()` using servicebus `queue_client`:
   - The notification method on POST should save the notification object and queue the notification id for the function to pick it up
2. Re-deploy the web app to publish changes

## Monthly Cost Analysis
Complete a month cost analysis of each Azure resource to give an estimate total cost using the table below:

| Azure Resource            | Service Tier      | Monthly Cost                  |
| ------------------------- | ------------      | ----------------------------- |
| *Azure Postgres Database* |  Standard         |  $16.09                       |
| *Azure Service Bus*       |  Basic            |  $0.05/1M operations          |
| *Azure Storage account*   |  Standard (Hot)   |  $0.05/1M operations          |
|  Azure App Service        |  Basic (B1)       |  $12.41                       |
|  Azure Function           |  Consumption Plan |  $0.20 per million executions |

## Architecture Explanation
In my project I don't change the structure of the project which show on udacity instruction and only use free and lowest cost I can choose.

Azure Functions can scale automatically based on the number of messages in the queue. This ensures that during peak loads, multiple instances of the function can process emails concurrently, preventing delays

By using Azure Service Bus Queue, the architecture achieves loose coupling between the web application and the email sending service. The web app can place email messages into the queue without worrying about the email service's availability or performance.

The use of Azure Functions to process messages from the Service Bus Queue allows for asynchronous email sending. This means the web app doesn't have to wait for the email to be sent, improving its responsiveness and user experience

- Azure Function: 

Azure Functions makes the app development process more productive, and lets you launch serverless applications on Microsoft Azure. It helps in processing data, coordinating with different systems for loT, integrating various processes and systems and building simple APIs and microservices.
It is autoscalable and allow to handle a huge peak of activity, the web app can't do this cost-effectively. This also, allows to do a division of concerns and create a service for email sending


- Azure services:
  
I choose cheapest options to save money because this project is small but you can change to other options for performance like B3 App service plan.
The Basic B1 tier has limited CPU and memory resources, which may not be sufficient for applications with high traffic or resource-intensive operations.
Scalability Issues: Inadequate scaling capabilities can lead to performance bottlenecks and increased latency, contributing to HTTP timeout errors during peak usage times.

- Service Bus:
  
Microsoft Azure Service Bus is a fully managed enterprise integration message broker. Service Bus can decouple applications and services.
Service Bus offers a reliable and secure platform for asynchronous transfer of data and state.

The Basic tier lacks some advanced features available in higher tiers, such as message sessions, transactions, and duplicate detection.
Message Processing Delays: If the service bus is under heavy load or if there are inefficient message processing practices, it can cause delays, leading to timeout errors for services dependent on timely message processing.

- Azure Postgres Database :
  
Performance Issues: The Standard tier might not provide sufficient performance for applications with high query volumes or complex transactions, leading to slow response times.
Timeouts: If the database queries take too long to execute due to performance limitations, it can cause HTTP timeout errors on the client side, especially for applications that require real-time data retrieval.




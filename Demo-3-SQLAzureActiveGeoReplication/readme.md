# Azure SQL Database Active Geo-Replication

This guide provides an introduction to Azure SQL Database Active Geo-Replication. In this demo, you will -

* Provision a Basic Azure SQL Database
* Scale the database up to Premium (P1)
* Enable Active Geo-Replication

## Pre-requisites

* Azure subscription
* Azure SDK 2.8 or higher
* Azure Powershell 1.0+
* SQL Server Management Studio

## Setup

*Estimated time: 30 minutes*

1. Go to [portal.azure.com](https://portal.azure.com).
2. Search for **SQL Database** and click to provision a new Azure SQL Server database.

  <img src="./media/prepstep1.png" style="max-height: 500px; max-width: 500px" />

3. The database is hosted within a *virtual* Azure SQL database server. Be sure to create a **v12** database server.

  <img src="./media/prepstep2.png" style="max-height: 500px; max-width: 500px" />

4. Change the **Source** database to the provided **AdventureWorksLT [V12]** sample database. 
5. Change the **Pricing tier** to **Premium P1**.

  <img src="./media/prepstep4.png" style="max-height: 500px; max-width: 500px" />

  > If you encounter an error during provisioning you can click the operation details to examine the error.
  
  <img src="./media/prepstep5.png" style="max-height: 500px; max-width: 500px" />
  
6. Once provisioned, add a client firewall rule so you can access your server from your local machine.

  <img src="./media/prepstep6.png" style="max-height: 500px; max-width: 500px" />

## Demo steps

*Estimated time: 8 minutes*

1. Go to [portal.azure.com](https://portal.azure.com).
2. Go to the resource group that contains the database that you provisioned during setup.
3. Click on the database **Settings** and click on **Pricing tier (scale DTUs)**.

  <img src="./media/step1.png" style="max-height: 500px; max-width: 500px" />

4. Show that we can change the pricing tier which changes DTUs and backup settings.

  <img src="./media/step2.png" style="max-height: 500px; max-width: 500px" />

  > There is a hidden slide at the end of the deck that covers DTUs. 
  > For more information on DTUs, take a look at [this article](https://azure.microsoft.com/en-us/documentation/articles/sql-database-service-tiers/#understanding-dtus).
  
5. Show the geo-replication is not currently configured.

  <img src="./media/step4.png" style="max-height: 500px; max-width: 500px" />

6. Click on **Configure Geo-Replication**. Choose a target region for the secondary read-only database. Based on [Azure paired regions](https://azure.microsoft.com/en-us/documentation/articles/best-practices-availability-paired-regions/), a secondary database server region will be recommended.

  <img src="./media/step5.png" style="max-height: 500px; max-width: 500px" />

7. Configure the Azure SQL Server server that will host your secondary database. Note that the secondary server is configured to use the same pricing tier as the primary server (Premium (P1)). This is required for active geo-replication.

  <img src="./media/step6.png" style="max-height: 500px; max-width: 500px" />

7. During provisioning, the secondary database status will first indicate **Initializing**, then shortly thereafter **Seeding** as the initial data set is replicated.

  <img src="./media/step7.png" style="max-height: 500px; max-width: 500px" />

8. Explain what is happening to the audience. we created a second, Azure SQL Database server and are replicating our primary database to the secondary region. The first location remains online during the replication. This operation typically takes about 10 minutes.
9. When complete, the secondary region will indicate **Readable** and the diagram will highlight the replication relationship.

  <img src="./media/step10.png" style="max-height: 500px; max-width: 500px" />

10. Go to the readable secondary server and add a client firewall rule so that you can access the secondary database from your local machine.
11. Open **SQL Server Management Studio** and connect to each database using the connection string provided in the portal.
12. Create a new query for each database and arrange the horizontally on the screen so both are visible.
13. Run the following SQL command against the primary database to add an initial record.

    ```sql
    insert into saleslt.customer (NameStyle, Title, FirstName, MiddleName,LastName, Suffix, CompanyName, SalesPerson, EmailAddress, Phone, PasswordHash, PasswordSalt, rowguid, ModifiedDate)
values(0, 'Mr.',  'Kirk',  'A.', 'Evans', NULL, 'GlobalDemo', 'adventure-works\pamela0','orlando0@adventure-works.com','245-555-0173', 'L/Rlwxzp4w7RWmEgXX+/A7cXaePEPcp+KwQhl2fJL7w=','1KjXYs4=', newid(), getdate())
    ```

14. Verify that the record was added to the primary database by running the following SQL query against the primary database.

    ```sql
  select * from saleslt.customer where CompanyName='GlobalDemo' 
    ```

15. Verify that the record was replicated to the secondary database by running the same SQL query against the secondary database.
16. Demonstrate Azure SQL Database failover using Azure Powershell using the following commands. Note that you will need to change the parameter values (database name, resource group name, etc.) to match your database name.

    ```powershell
  $database = Get-AzureRMSqlDatabase `
              –DatabaseName "[Database_Name]" `
              –ResourceGroupName "[Resource_Group_Name]" `
              –ServerName "[Server_Name]" 

  $database | Set-AzureRMSqlDatabaseSecondary `
              –Failover `
              –PartnerResourceGroupName "[Resource_Group_Name]"
    ```
    
## Clean up

To clean up this environment simply delete the resource group that was created during setup. If you intend on keeping the database, stop replication between them and change the database tier to **Basic**. This will avoid unnecessary charges to your bill (or potentially exhausting your MSDN credit).

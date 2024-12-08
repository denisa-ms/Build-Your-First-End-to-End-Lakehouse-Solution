# Exercise 3 - Collaborate inside Notebooks and share Lakehouse. Use SQL Endpoint and SSMS

> [!NOTE]
> Timebox: 30 minutes
> 
> [Back to Agenda](./../README.md#agenda) | [Back to Exercise 2](./../exercise-2/exercise-2.md) | [Up next Exercise 4](./../exercise-4/exercise-4.md)
> #### List of exercises:
> * [Task 3.1 Retrieve Lakehouse SQL Analytics Endpoint Connection String](#task-31-retrieve-lakehouse-sql-analytics-endpoint-connection-string)
> * [Task 3.2 Connect to a Fabric SQL Endpoint Using SQL Server Management Studio (SSMS)](#task-32-connect-to-a-fabric-sql-endpoint-using-sql-server-management-studio-ssms)
> * [Task 3.3 Execute T-SQL Queries on Lakehouse Delta Tables](#task-33-execute-t-sql-queries-on-lakehouse-delta-tables)
> * [Task 3.4 Sharing a Lakehouse](#task-34-sharing-a-lakehouse)
> * [Task 3.5 Sharing a Notebook for Collaboration](#task-35-sharing-a-notebook-for-collaboration)


The **SQL Analytics Endpoint** of a Fabric Lakehouse Offers a SQL-based experience for analyzing data in lakehouse delta tables using T-SQL language, with features like saving functions, generating views, and applying SQL security.

When a lakehouse is shared, users are automatically granted Read permission, which applies to the lakehouse itself, the linked SQL endpoint, and the default semantic model. Beyond this standard access, users may also be granted:

-   **ReadData** permission for the SQL endpoint, enabling data access without the enforcement of SQL policies.
-   **ReadAll** permission for the lakehouse, allowing comprehensive data access via Apache Spark.
-   **Build** permission for the default semantic model, permitting the creation of Power BI reports utilizing this model
  
---

# Task 3.1 Retrieve SilverCleansed Lakehouse SQL Analytics Endpoint Connection String

The goal is to obtain the SQL connection string for your SilverCleansed Lakehouse's SQL analytics endpoint, which is crucial for connecting and querying your data through SQL-based tools.

1. **Access the Analytics Endpoint**:
   - Go to your workspace and find the SilverCleansed Lakehouse SQL analytics endpoint.
   - Click on `More options` (usually represented by three dots or an ellipsis icon) associated with the analytics endpoint.

2. **Copy the SQL Connection String**:
   - From the available options, select `Copy SQL connection string`.
   - This action copies the connection string to your clipboard, ensuring you have the necessary information to establish a SQL connection.
     ![Copy Connection String](../screenshots/3/CopyConnectionString.png)

3. **Utilize the Connection String**:
   - With the connection string now on your clipboard, you can use it to connect to your Lakehouse SQL analytics endpoint.
   - Open a database tool of your choice, such as SQL Server Management Studio (SSMS) or Azure Data Studio.
   - Start a new connection dialogue, paste the connection string into the appropriate field, and follow the prompts to establish a connection.

Ensure that you handle the connection string securely, as it provides access to your data within the Lakehouse. Avoid sharing it openly or storing it in unsecured locations. If you encounter any issues while copying or using the connection string, review the settings and permissions within your Lakehouse workspace or consult the relevant documentation.

---

# Task 3.2 Connect to a Fabric SQL Endpoint Using SQL Server Management Studio (SSMS)
> [!TIP]
> If you are interested in lineage and connecting through Azure Data Studio, [proceed to this additional exercise](../exercise-extra/extra.md#lineage).
 
The goal of this task is to establish a connection to a Fabric SQL Endpoint using SQL Server Management Studio (SSMS), enabling you to query and manage your data directly from SSMS. [Download the latest generally available (GA) version of SQL Server Management Studio (SSMS) 20.0 (485 MB)](https://aka.ms/ssmsfullsetup)

1. **Open SQL Server Management Studio**:
   - Launch SSMS on your computer. The `Connect to Server` window should automatically appear upon opening the application. If you're already in SSMS but not connected, navigate to Object Explorer, click `Connect`, and then select `Database Engine`.

2. **Enter Server Details**:
   - In the `Server name` field of the connection window, paste the SQL connection string you previously copied. This string should correspond to your Fabric SQL Endpoint.

3. **Authentication**:
   - For the authentication method, select `Microsoft Entra Password` from the options. This ensures a secure connection utilizing modern authentication methods.

    ![password](../screenshots/3/pwd.jpg)

4. **Enter User Credentials**:
   - In the authentication window that appears, enter your workshop user email or your enterprise email ID. Follow the prompts to complete the multifactor authentication process.

5. **Explore the Lakehouse**:
   - Once connected, the Object Explorer panel in SSMS will show the connected Lakehouse. You can expand the server node to view the databases (lakehouses) and navigate through tables, views, and other objects available for querying.

> [!IMPORTANT]
> Remember to handle sensitive information, such as connection strings and credentials, securely. Ensure that you have the correct permissions to access the data and the SQL endpoint. If you encounter any connection issues, verify your connection string and authentication details. Also, check your network settings and firewall rules that may block the connection to the Fabric SQL Endpoint.

---

# Task 3.3 Execute T-SQL Queries on Lakehouse Delta Tables

Execute a series of T-SQL queries on the Lakehouse Delta tables, particularly focusing on data analysis of the NYC Taxi table from the `silvercleansed` database. These queries will help you understand data aggregation, view creation, and basic SQL operations within your Lakehouse environment.

1. **Count Rows in the NYC Taxi Table**:
   - Execute the following SQL query to get the total number of rows in the `green201501_cleansed` table:
     ```sql
     SELECT COUNT(*)
     FROM [silvercleansed].[dbo].[green201501_cleansed];
     ```

2. **Calculate Average Fare and Tip Amount**:
   - Run the below query to calculate the average fare and tip amount from the same table:
     ```sql
     SELECT ROUND(AVG([fareAmount]),2) AS [Average Fare], 
     ROUND(AVG([tipAmount]),2) AS [Average Tip] 
     FROM [silvercleansed].[dbo].[green201501_cleansed];
     ```

3. **Aggregate Fares by Passenger Count**:
   - Use the following query to get the total and average fares grouped by the passenger count, ordered by average fares in descending order:
     ```sql
     SELECT DISTINCT [passengerCount], 
     ROUND(SUM([fareAmount]),0) as TotalFares,
     ROUND(AVG([fareAmount]),0) as AvgFares
     FROM [silvercleansed].[dbo].[green201501_cleansed]
     GROUP BY [passengerCount]
     ORDER BY AvgFares DESC;
     ```

4. **Compare Tipped Versus Not Tipped Trips**:
   - Execute this query to compare the number of trips where a tip was given versus not:
     ```sql
     SELECT tipped, COUNT(*) AS tipFreq FROM (
       SELECT CASE WHEN (tipAmount > 0) THEN 1 ELSE 0 END AS tipped, tipAmount
       FROM [silvercleansed].[dbo].[green201501_cleansed]
       WHERE [lpepPickupDatetime] BETWEEN '20150101' AND '20151231') tc
     GROUP BY tipped;
     ```

5. **Create a View for Average and Total Fares by Passenger Count**:
   - Run the following SQL command to create a view based on the SQL used in step 3:
     ```sql
     CREATE VIEW [dbo].[viGetAverageFares]
     AS 
     SELECT DISTINCT [passengerCount], 
     ROUND(SUM([fareAmount]),0) as TotalFares,
     ROUND(AVG([fareAmount]),0) as AvgFares
     FROM [silvercleansed].[dbo].[green201501_cleansed]
     GROUP BY [passengerCount];
     ```

6. **Query the Newly Created View**:
   - Lastly, retrieve data from your newly created view to ensure it's been set up correctly:
     ```sql
     SELECT * FROM [silvercleansed].[dbo].[viGetAverageFares];
     ```

> [!IMPORTANT]
> Make sure you have the proper permissions to execute these queries and create views within the Lakehouse. Pay close attention to the syntax and database structure to ensure accurate results. Document any interesting findings or anomalies encountered during the analysis for further investigation or discussion.

---

# Task 3.4 Sharing a Lakehouse

Learn how to share a Lakehouse with team members or stakeholders within your workspace, ensuring they have the appropriate level of access.

1. **Navigate to Your Lakehouse**:
   - In your Workspace, locate the Lakehouse you wish to share.
   - Click the **Share** button located next to the lakehouse name.
     ![Lakehouse Share](https://github.com/ekote/Build-Your-First-End-to-End-Lakehouse-Solution/assets/63069887/f33d4f80-d24e-4804-b81f-ea4fd2f3188d)

2. **Configure Sharing Settings**:
   - In the Sharing dialog, enter the name or email address of the individuals you wish to share the Lakehouse with.
   - Assign the appropriate permissions by checking the relevant boxes. By default, sharing the Lakehouse grants access to the lakehouse, the associated SQL endpoint, and the default semantic model.
   
   ![Lakehouse Sharing Dialog](https://github.com/ekote/Build-Your-First-End-to-End-Lakehouse-Solution/assets/63069887/b7d04784-d5c6-44d9-accb-4e7119d6fea8)

3. **Notification Settings**:
   - If you want to notify the recipients via email, check the **`Notify recipients by mail`** option.
   - Include an optional message to provide context or instructions for the recipients.

4. **Finalize Sharing**:
   - Once you've configured the sharing settings and notification preferences, click **Grant** to finalize sharing the Lakehouse.

> [!IMPORTANT]
> Ensure that you only share the Lakehouse with individuals who require access and have the appropriate level of permissions according to their needs and roles. Review and adhere to your organization's data sharing and privacy policies when sharing Lakehouse resources. Keep track of who has access to the Lakehouse for future reference and security compliance.

---

# Task 3.5 Sharing a Notebook for Collaboration

Learn how to share a notebook with team members within your workspace, allowing for collaboration with specified permissions.

1. **Open the Notebook**:
   - Navigate to the notebook that you wish to share.
   - Click on the **Share** button located on the notebook toolbar.
     ![Share Button](https://github.com/ekote/Build-Your-First-End-to-End-Lakehouse-Solution/assets/63069887/496e0f19-3d63-4e6f-9698-1adcbdf2f052)

2. **Set Permissions**:
   - In the sharing settings, select the category of **people who can view this notebook**.
   - Assign appropriate permissions by selecting from **Share**, **Edit**, or **Run**. This will determine what recipients can do with the notebook.
     ![Set Permissions](https://github.com/ekote/Build-Your-First-End-to-End-Lakehouse-Solution/assets/63069887/f6674e9e-791e-4f7b-84b6-43b2140e0e6d)

3. **Share the Notebook**:
   - After setting the permissions, click **Apply**.
   - You can then choose to send the notebook directly to your team members or copy the link and distribute it manually. Recipients will be able to access the notebook according to the permissions you have set.
     ![Share Options](https://github.com/ekote/Build-Your-First-End-to-End-Lakehouse-Solution/assets/63069887/0a097d72-0a5e-4617-8920-6fd0439d8cad)

4. **Manage Notebook Permissions**:
   - For additional permission settings or to update access, navigate to the Workspace item list.
   - Click **More options** next to your notebook and select **Manage permissions**. Here, you can modify who has access and what level of access they hold.
     ![Manage Permissions](https://github.com/ekote/Build-Your-First-End-to-End-Lakehouse-Solution/assets/63069887/b37e8de8-36d8-4a4b-accb-4b67c901f26a)


> [!NOTE]
> Be mindful of the data and information contained in the notebook when sharing, ensuring that only the appropriate parties receive access. Review your organization’s policies on data sharing and collaboration to comply with security and privacy standards. Document any issues or challenges encountered during the sharing process for future reference or to seek assistance.

---


> [!IMPORTANT]
> Once completed, go to [next exercise (Exercise 4)](./../exercise-4/exercise-4.md). If time permits before the next exercise begins, consider continuing with [extra steps](../exercise-extra/extra.md).

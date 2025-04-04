# Cohort Management App – Detailed User Documentation

We developed the Cohort Management App using Streamlit and the Snowflake Native App framework so that you can securely create, filter, save, and schedule data cohorts in your own Snowflake account. In addition, the app supports geospatial data analysis and visualization. This documentation offers clear, step‐by‐step instructions for every screen and feature, guiding you through the entire workflow.

Also, here are a **[step-by-step instructions](cohort_builder_walkthough.md)** on how to create you first cohort.

---

## 1. Home Screen

The Home Screen is your starting point. Here, you connect to Snowflake and access the app’s main features. It helps you manage warehouses, monitor your connection status, and navigate to other sections of the app. This screen ensures your environment is properly set up before you begin selecting data and creating cohorts.
It also displays a brief introduction for the three core functions: building cohorts, viewing existing cohorts, and scheduling updates.

#### Snowflake Connection

- Your connection to Snowflake is established automatically.
- The screen shows your connection status and alerts you if there is an issue.

#### Warehouse Management

- **View Warehouse:** You can view the currently active Snowflake warehouse used for creating tasks and dynamic tables.
- **Create or Drop Warehouses:**
  - If no warehouse is available, a form appears so that you can create one by entering the warehouse name, size, and auto-suspend time.
  - You also have the option to drop an existing warehouse using the provided button.
- **Privileges Check:**
  - The app verifies that you have the necessary privileges.
  - If any privileges are missing, it offers clear guidance on how to grant them.

#### Navigation Guidance

- A sidebar provides buttons and links to help you navigate between screens.
- Each section on the Home Screen includes a summary that explains its purpose.

---

## 2. Grant Access Screen (Part of Select Dataset)

Before you begin working with your data, you must grant the app permission to access your tables in Snowflake. This step ensures the app can retrieve the data needed to build cohorts. While the Home Screen allows you to verify your Snowflake connection and warehouse status, this screen focuses on managing access permissions. Proper access is essential for secure data operations and uninterrupted query execution.

### Options for Granting Access

You can grant access to your data tables in one of two ways:

### SQL Grant Statement (Preferred for Production)

For production environments, use the SQL Grant Statement method. This method requires you to explicitly grant the necessary privileges using SQL commands, ensuring that only authorized access is provided. If needed, you must directly revoke privileges.

**Example Queries:**

```sql
GRANT USAGE ON DATABASE <YOUR_DATABASE> TO APPLICATION COHORT_BUILDER_APP;
GRANT USAGE ON SCHEMA <YOUR_DATABASE>.<YOUR_SCHEMA> TO APPLICATION COHORT_BUILDER_APP;
GRANT SELECT ON TABLE <YOUR_DATABASE>.<YOUR_SCHEMA>.<YOUR_TABLE> TO APPLICATION COHORT_BUILDER_APP;
```
You can also grant access to all tables in the schema:
```sql    
GRANT USAGE ON DATABASE <YOUR_DATABASE> TO APPLICATION COHORT_BUILDER_APP;
GRANT USAGE ON SCHEMA <YOUR_DATABASE>.<YOUR_SCHEMA> TO APPLICATION COHORT_BUILDER_APP;
GRANT SELECT ON ALL TABLES IN SCHEMA <YOUR_DATABASE>.<YOUR_SCHEMA> TO APPLICATION COHORT_BUILDER_APP;   
```
These commands provide the app with the permissions required to query your database, schema, and table. For more details, please refer to Snowflake's documentation on GRANT statements.

**Dataset Selection**

Select a database from the dropdown menu, which reloads the schema list and resets dependent options; then, choose a schema to refresh the table list and finally select a table from the list



### Reference Method (For Quick Testing)

Alternatively, you can use the built-in Security dialog to grant temporary access via table references. This method is suitable for testing or rapid prototyping, although it may cause cohorts to fail if table references change over time.

**How It Works:**  
- **Radio Button Choice:**  
  When you reach the dataset selection screen, a radio button asks, “Which method did you use to grant table access?” Select either “Grant Statement” or “Reference.”
  
- **Impact on Table Selection:**  
  - Using the SQL Grant Statement method displays your own database, schema, and table lists based on the privileges you granted.
  - Using the Reference method displays a list of pre-approved (referenced) tables.
  
- **Integration with Home Screen:**  
  The Home Screen shows your Snowflake connection and warehouse details, confirming that you have the required privileges. If any are missing, the app guides you with the provided sample queries.

This approach is recommended for test and temporary cohorts. 
Once you change the selection in the Security section, existing references no longer work and created cohorts will fail.

**Automatic Data Loading:**  
After you select a table, the app automatically loads a preview of the table data along with metadata (such as column names, data types, and comments) and summary statistics (like total row count and table size). Callback functions clear previous selections and reset filters.

#### Data Preview & Summary Cards

#### Summary Cards

Three cards display key metrics:  
- **Rows:** The number of records (formatted in K/M for large numbers).  
- **Memory:** The size of the table.  
- **Columns:** The number of columns in the dataset.

These cards help you quickly understand the scale of your data.

#### Data Grid Preview

A table view displays a sample of the dataset, allowing you to verify the data before proceeding with filtering and cohort creation.

---

## 3. Data Dictionary Screen

The Data Dictionary Screen provides a detailed view of your dataset’s structure. Here, you can review column metadata and set up filters, making it easier to determine which fields are most relevant for your analysis. This screen is essential for tailoring your filtering strategy by displaying available data types and filter options for each column.

### A. Column Metadata

**Display of Metadata:**  
You can view a list of all columns in the selected table, including each column's name, data type (e.g., TEXT, NUMBER, TIMESTAMP), and any existing comments or descriptions. The app retrieves this metadata from the `INFORMATION_SCHEMA` or from a table description if you use references.

### B. Filter Options Setup

**Primary vs. Secondary Filters:**  
- **Primary Filters:** Quickly reduce the dataset by targeting columns with a high filtering impact.  
- **Secondary Filters:** Fine-tune the cohort after applying primary filters.

**Filter Types Available:**  
- **Dropdown:** Choose a single option for categorical data (SQL logic: `column = 'value'`).
- **Multi-Select Dropdown:** Choose multiple values (SQL logic: `column IN ('value1', 'value2', …)`).
- **Boolean Filter:** Toggle between true/false values (SQL logic: `column = true/false`).
- **Slider / Range Filter:** Select a range of values (e.g., numeric or date ranges) (SQL logic: `column BETWEEN min AND max`).
- **Advanced Date Filter:** Choose options like “between,” “greater than or equal,” or “less than or equal” for precise date range filtering.
- **Advanced Numeric Filter:** Similar to the date filter, but designed for numeric values.
- **Text Filter:** Apply conditions such as “Contains,” “Does not contain,” “Equals,” “Does not equal,” “Begins with,” “Ends with,” “Blank,” or “Not blank” (SQL logic uses `LIKE`, `NOT LIKE`, or equality comparisons).
- **Geo-Radius Filter (Geospatial):** If your table contains geospatial data (such as columns of type “Long/Lat”), you can define a central point and a radius (SQL logic uses spatial functions like `ST_DISTANCE` and `ST_MAKEPOINT`).

### C. LLM Enrichment (Optional)

**LLM Toggle in Sidebar:**  
You can enable LLM enrichment by selecting the checkbox labeled “Use LLM for Data Dictionary?” in the sidebar. When enabled, the app uses the `mistral-large` model to:  
- Automatically generate descriptive text for each column,  
- Suggest whether a column should be a primary filter, and  
- Recommend an appropriate filter type.

This process takes about a minute and updates the metadata on screen.


### D. Geospatial Visualization and Filtering

This section allows you to analyze location-based data within your cohorts. You can apply geospatial filtering and visualization to add a spatial dimension to your analysis, which is particularly useful when your dataset includes geographical coordinates.

#### Detection of Geospatial Data

If your table contains geospatial data (for example, columns labeled “Long/Lat”), you can access geospatial filtering and visualization options. At the moment application supports only table with two separate columns for Latitude and Longitude. You can assign which columns in your table have that values.
You should select if you need this data for visualization only or for filtering your cohort. Also choose if this filter should be primary or secondary.

---

## 4. Build Cohort Screen

On the Build Cohort Screen, you define and create a new data cohort. You apply your chosen filters to narrow down the dataset and generate a customized SQL query. This screen bridges data exploration and actionable output by allowing you to review, adjust, and save your cohort for future analysis.

### A. Applying Filters

#### Primary Filters Sidebar

Use the controls on the sidebar to apply primary filters. You can use dropdowns, sliders, and text fields to immediately narrow down your dataset.

#### Secondary Filters Display

After you apply primary filters, additional options appear to help you further refine your dataset.

#### Filter Types Available

- **Dropdown:**
  - A single-choice list for categorical data.
  - SQL Logic: `column = 'value'`.
  
- **Multi-Select Dropdown:**
  - Choose multiple values.
  - SQL Logic: `column IN ('value1', 'value2', ...)`.
  
- **Boolean Filter:**
  - Toggle true/false values.
  - SQL Logic: `column = true/false`.
  
- **Slider / Range Filter:**
  - Select a range of values (e.g., numeric or date ranges).
  - SQL Logic: `column BETWEEN min AND max`.
  
- **Advanced Date Filter:**
  - Options include “between,” “greater than or equal,” or “less than or equal.”
  - Allows precise date range filtering.
  
- **Advanced Numeric Filter:**
  - Similar options as date filters but for numeric values.
  
- **Text Filter:**
  - Conditions include “Contains,” “Does not contain,” “Equals,” “Does not equal,” “Begins with,” “Ends with,” “Blank,” or “Not blank.”
  - SQL Logic uses `LIKE`, `NOT LIKE`, or equality comparisons.
 
- **Geo-Radius Filter**
  - Define a central point by entering a longitude and latitude.
  - Specify a radius (in kilometers).
  - The app generates a SQL condition using spatial functions (such as `ST_DISTANCE` and `ST_MAKEPOINT`) to filter records within the specified radius.


### B. SQL Query Generation

#### SQL Preview
As you apply filters, the app generates a SQL query and automatically formats it (using `sqlparse`) for easier review. You can see how each filter is converted into SQL:  
- For a dropdown, the query includes `column = 'value'`.  
- For a multi-select dropdown, it includes `column IN ('value1', 'value2', …)`.  
- For a range, it uses `BETWEEN min AND max`.  
- For text conditions, it uses `LIKE` or `NOT LIKE`.  
- For geo-radius filters, it uses spatial functions to compare distances.

#### Cohort Definition 
Review the generated SQL to ensure it meets your intended filter logic. Then, use this query to build a cohort table in your Snowflake environment.

#### Map Visualization

When geospatial data is available, an interactive map is displayed:  
- Data points within the defined radius are plotted.
- Hovering over a point displays additional location details in tooltips.

### C. Saving the Cohort

**Save Option:**  

 - When you are satisfied with your cohort, click the save button. 
 - The app stores the cohort as either a scheduled or static table based on your configuration.
 - If your cohort already saved or just loaded you can update it after making some changes to the filters.
 - If you scheduled your cohort as a Dynamic table, updating the Cohort will effectively delete the cohort table and require you to re-create your schedule.
---

## 5. Schedule Cohort Screen

On the Schedule Cohort Screen, you set up automation settings for your saved cohorts. This allows you to schedule periodic updates so that your cohort remains current and aligned with your business needs.

### A. Scheduling Options

**Types of Schedules:**  
- **Dynamic (Auto-Updating) Cohorts:** The cohort table refreshes automatically when new data is available.
- **Snapshot Cohorts:** The app captures data at a specific point in time.
- **One-Time Cohorts:** The app runs the cohort query once without recurring updates.

**Configuration Options:**  
- **Table Name:** Set a name for the cohort's table.
- **Lag Time:** Set a delay between data updates.
- **Refresh Mode:** Choose how frequently the data is refreshed.
- **Other Options:** Additional parameters let you control the scheduling behavior.

### B. SQL Preview & Confirmation

**Query Preview:**  
Review the SQL query preview to see how the scheduling options modify the query.

**Final Confirmation:**  
Once you are satisfied, confirm the schedule. The app then creates a task in Snowflake to execute the cohort query as scheduled.

---

## 6. Existing Cohorts Screen

The Existing Cohorts Screen provides an overview of all your saved cohorts. Here, you can load your cohorts, to review, modify, delete, or use them for further analysis.

### A. Cohort List

**Overview:**  
View a list of saved cohorts with key details such as the cohort name, associated SQL query, row count, and status.

**Selection:**  
Click on a cohort to load its details, including the full definition and applied filters.

### B. Cohort Actions

- **Modify:** Edit the filter criteria or scheduling options at the Build Cohort screen after loading and update the SQL query if needed.
- **Delete:** Remove a cohort that is no longer required.
- **Reload:** Load an existing cohort to update its filters or review its data.
- **Geospatial Data Visualization (if applicable):** If your cohort includes geospatial data, interact with an interactive map (built with PyDeck) that displays data points and tooltips to help you identify geographic trends.


---

## 7. Technical Details & Logic Behind Options

### Filter Logic

**Filter Generation Functions:**  
- **Non-Text Filters:** The app creates a corresponding SQL clause based on the filter type (dropdown, multi-select, boolean, slider/range, advanced date/numeric).
- **Text Filters:** Conditions such as “Contains” or “Equals” are translated into SQL using `LIKE` or equality operators.
- **Geo-Radius Filters:** Spatial functions compute distances to filter records based on the specified radius.

**State Management:**  
The app uses Streamlit’s session state to track your selections and refresh data when changes occur. Callback functions clear outdated state variables whenever you select a new database, schema, or table.

**LLM Enrichment:**  
If enabled, the app uses the LLM (`mistral-large`) to generate enhanced descriptions, suggest primary filters, and recommend appropriate filter types. This feature makes the Data Dictionary more user-friendly, especially if you are not deeply familiar with the data.

**Security & Performance:**  
- **Secure Data Handling:** All operations run within your Snowflake environment, ensuring that your data remains secure.
- **Caching:** Streamlit caching speeds up data loading.
- **SQL Formatting:** The app formats SQL queries before they are displayed for improved readability.

---
If you need some visual examples, check our **[step-by-step instructions](cohort_builder_walkthough.md)** on how to create you first cohort.

Happy cohorting!

_This app is the result of a close collaboration between the Snowflake team and BlueCloud, and a tribute to the hard work of the Snowflake Solution Innovation Team. Their dedication and expertise allowed us to harness the power of the Snowflake Data Cloud, creating a tool that gives businesses valuable insights and supports digital transformation._


# 📘 Databricks Masterclass

A comprehensive guide based on the Databricks Masterclass, detailing the concepts and skills I acquired, including working with the Databricks File System (DBFS), accessing Azure Data Lake securely, utilizing Delta Lake for efficient data processing, and implementing streaming pipelines with Auto Loader.



---

## 📁 Databricks File System (DBFS)
- DBFS is a distributed file system native to Databricks.
- Provides abstraction to treat storage like traditional filesystems (e.g., `/FileStore/raw.file.csv`).
- Connects securely to Delta Lake via URL-based paths.

---

## 🔐 Hardcoded Access: Databricks to Azure Data Lake
- **Mounting** is used for persistent storage access.
- **Service Principal** grants programmatic access.
- **Steps to connect:**
  - Go to **Entra ID → App Registration**
  - Assign **Storage Blob Data Contributor** role
  - Gather:
    - `Application_ID`
    - `Tenant_ID`
    - `Client Secret` or Certificate

---

## 🛠️ Databricks Utilities
### 🔎 File System Checks
```python
dbutils.fs.ls("abfss://source@storageaccount.dfs.core.windows.net/")
```

### 🧮 Parameterization with Widgets
```python
dbutils.widgets.text("p_name", "girald")
dbutils.widgets.get("p_name")
```

---

## 🔑 Secure Secrets with Azure Key Vault
### 🏗️ Setup Process
1. **Azure Portal → Key Vault → Create**
2. Access Configuration → Vault Access Policy
3. IAM → Add Role: `Key Vault Administrator`
4. Create Secret (`app-secret`) with required value

### 🔗 Link Key Vault to Databricks
1. Navigate to Databricks URL → `#secrets/createScope/`
2. Fill in:
   - **Scope Name**: `giraldScope`
   - **Manage Principal**: `Creator`
   - **DNS Name** and **Resource ID** from Key Vault → Properties

### 🔍 Accessing Secrets
```python
dbutils.secret.list(scope="giraldscope")
dbutils.secret.get(scope="giraldscope", key="app-secret")
```

---

## 📥 Data Reading with PySpark
```python
df_sales = spark.read.format("csv") \
    .option("header", True) \
    .option("inferSchema", True) \
    .load("abfss://source@storagenameaccount.dfs.core.windows.net/")
```

---

## 🔄 PySpark Transformations
```python
from pyspark.sql.functions import *
from pyspark.sql.types import *
```

### ➗ Split a Column
```python
df_sales.withColumn("testdata", split(col("testcolumn"), " ")).display()
```

### ➕ Add Constant
```python
df_sales.withColumn("testdata", lit("your_value"))
```

### 🔁 Change Data Type
```python
df_sales.withColumn("testdata", col("testdata").cast(StringType())).display()
```

---

## 💾 Delta Lake: Format & Transactions
- Delta Lake supports transactional logs: `.json`, `.crc`
- `%run` includes external notebook sessions
```python
%run "/foldernameofworkspace/filenameofnotebook"
```

### 📝 Save as Delta
```python
df_sales.write.format("delta") \
    .mode("append") \
    .option("path", "abfss://foldername@storagename.dfs.core.windows.net/") \
    .save()
```

### 🐝 Use Catalog
```sql
USE CATALOG hive_metastore;
```

---

## 🧠 Delta Lake Advanced
### 📜 Versioning
```sql
DESCRIBE HISTORY table_name;
```

### 🕒 Time Travel
```sql
RESTORE TABLE table_name TO VERSION AS OF 1;
```

### 🧹 Vacuum (Cleanup)
```sql
VACUUM table_name;
```
- Deletes old files (default: 7 days)

### ⚙️ Optimize
```sql
OPTIMIZE table_name;
```

### 🔀 Z-Order Clustering
```sql
OPTIMIZE table_name ZORDER BY (id);
```

---

## 🌊 Auto Loader: Streaming Data
```python
df = spark.readStream.format("cloudFiles") \
    .option("cloudFiles.format", "parquet") \
    .option("cloudFiles.schemaLocation", "abfss://aldestination@datalake.dfs.core.windows.net/checkpoint") \
    .load("abfss://alsource@datalake.dfs.core.windows.net")

# Write stream

df.writeStream.format("delta") \
    .option("checkpointLocation", "abfss://aldestination@datalake.dfs.core.windows.net/checkpoint") \
    .option("mergeSchema", "true") \
    .trigger(processingTime="5 seconds") \
    .start("abfss://aldestination@datalake.dfs.core.windows.net/data")
```

---

## ⛓️ Workflows (Jobs)
- Automate notebook runs
- Schedule pipelines and manage dependencies
- Pass parameters between notebooks

---
Happy building! 🚀

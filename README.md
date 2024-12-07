## Prerequisites
1. Ensure you have the following installed:
   - Python
   - Docker
   - Node.js (You can alternatively use bun, pnpm, etc.)
<br>
2. Clone the repository:
```bash
git clone [repository_url]
cd [repository_name]
```

## Step 1: Start the Hive Server
### Step 1.1: Start Hive Server
Go to `docker-hive` directory, and run `docker-compose up -d` to run the containers

### Step 1.2: Get HDFS ready
Run to connect to the namenode
```bash
docker exec -it <namenode_container_id> /bin/bash
```

then run inside the name-node container
```bash
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -chmod -R 777 /user/hive/warehouse
```

### Step 1.3: Transfer your data (.csv/.ssv/.tsv) into HDFS
Run to copy data from your local to your `hive-server` container
```bash
docker cp /path/to/yourfile.csv <hive-server_container_id>:/tmp/yourfile.csv
```

### Step 1.4: Login to hive-container and put the copied data into HDFS
Run in a new terminal
```bash
docker exec -it <hive-server_container_id> /bin/bash
```

Put the data in HDFS
```bash
hdfs dfs -mkdir -p /user/hive/warehouse/your_table
hdfs dfs -put /tmp/yourfile.csv /user/hive/warehouse/your_table/
```

### Step 1.4: Create the table in Hive
First connect to the hive interface
```bash
$ hive
```

Create the table by running the SQL query. Make sure to edit the query as per your need.
```sql
CREATE EXTERNAL TABLE your_table (
    column_1 DATE,
    column_2 FLOAT,
    column_3 FLOAT,
    column_4 FLOAT,
    column_5 FLOAT
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
    "serialization.format" = ",",
    "field.delim" = ","
)
STORED AS TEXTFILE
LOCATION '/user/hive/warehouse/your_table/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

### Step 1.5: Connect to the hive server from backend server
After doing the step 1.4 your table and data set up. You can connect to the hive server after this. You can use the following python snippet for reference.

```python
from pyhive import hive

conn = hive.Connection(
    host='localhost', 
    port=10000, 
    username='your_user', 
    database='default'
)

cursor = conn.cursor()
cursor.execute('SELECT * FROM discover_findata LIMIT 10')
results = cursor.fetchall()

for row in results:
    print(row)
```
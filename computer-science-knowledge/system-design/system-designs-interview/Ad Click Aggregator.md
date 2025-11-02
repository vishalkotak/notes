Reference: https://www.hellointerview.com/learn/system-design/problem-breakdowns/ad-click-aggregator

#### Extensions

### Mapping for Hot Keys

Alright — you are asking **exactly the right questions** to design **real production-quality streaming systems**. 🔥  
Let’s go into **serious depth** about **broadcast variables**, how they work, and how to **refresh them dynamically** without restarting your Spark job.

---

# 🧠 Step 1: What is a Broadcast Variable in Spark?

- A **broadcast variable** is a way to send **read-only data** from the **driver** to **all executors**.
    
- Spark **transfers** the data **once** (optimized with compression and block transfer).
    
- After that, all Spark tasks **locally cache** the broadcasted object.
    
    - So **no repeated network transfer** — very fast memory lookup.
        

✅ Used for things like:

- Lookup tables (e.g., ad_id splits)
    
- Machine learning models
    
- Configurations
    

---

# 🏗 Step 2: How to create and use a Broadcast Variable

**1. Initial broadcast at job start:**

```python
# Driver code
mapping = {"ad_123_0": "ad_123", "ad_123_1": "ad_123"}
broadcast_var = spark.sparkContext.broadcast(mapping)
```

**2. Use it inside transformations:**

```python
from pyspark.sql.functions import udf

def remap_key(split_key):
    return broadcast_var.value.get(split_key, split_key)

remap_key_udf = udf(remap_key)

df.withColumn("base_key", remap_key_udf(df.split_key))
```

✅ Each task uses the `broadcast_var.value` to do **fast local lookup**.

---

# 🔥 Step 3: Now the real question — how to **refresh** a Broadcast Variable when the mapping changes?

**Problem**:

- Spark does **NOT** automatically update broadcast variables after they're created.
    
- Broadcast variables are **immutable**.
    
- You must **manually re-broadcast** a new copy **AND** **update all tasks** to use the new one.
    

---

# 🚀 Step 4: How to Refresh a Broadcast Variable Dynamically

Here’s the general pattern:

|Step|What you do|
|:--|:--|
|1|Pull the **latest mapping** from your source (DB, S3, service, etc.)|
|2|**Unpersist** the old broadcast variable to free memory.|
|3|**Broadcast** the new mapping.|
|4|**Switch** the code (UDF, transformations) to use the **new broadcast variable**.|

---

# 🛠 Code Pattern Example

```python
# Global holder for broadcast variable
broadcast_var = None

def update_mapping():
    global broadcast_var
    
    # Pull updated mapping (e.g., from DB, API, file, etc.)
    updated_mapping = get_latest_mapping()
    
    # Remove old broadcast variable
    if broadcast_var:
        broadcast_var.unpersist()
    
    # Broadcast new variable
    broadcast_var = spark.sparkContext.broadcast(updated_mapping)

def remap_key(split_key):
    return broadcast_var.value.get(split_key, split_key)

remap_key_udf = udf(remap_key)

# Spark Streaming Query
df = spark.readStream.format("kafka").option(...).load()

# Initially broadcast
update_mapping()

query = (
    df.withColumn("base_key", remap_key_udf(df.split_key))
      .writeStream
      .format("console")
      .start()
)

# In a separate thread, keep refreshing mapping every few minutes
import threading
import time

def periodic_refresh():
    while True:
        time.sleep(300) # 5 minutes
        update_mapping()

refresh_thread = threading.Thread(target=periodic_refresh, daemon=True)
refresh_thread.start()
```

✅ Now your Spark job **stays live**, but **gets updated mappings** every few minutes!

---

# ⚡ Very important details

- **You must call `unpersist()`** on the old broadcast variable to avoid memory leaks.
    
- If you are inside a **UDF**, Spark automatically reads the latest `broadcast_var` because **UDF closures capture variables by reference** in Python.
    
- In Java/Scala, you sometimes need **to re-register** the UDF or refresh it manually.
    
- Refreshing **too often** (like every few seconds) is bad — Spark still has to push new broadcast to all executors.
    

---

# ✨ Summary

|Question|Answer|
|:--|:--|
|Can you update a broadcast variable?|**Yes, manually by rebroadcasting**|
|How to refresh mapping dynamically?|**Unpersist old, broadcast new, update UDF reference**|
|How often should you refresh?|Every few minutes (not seconds), unless mapping changes very frequently|
|Is this production-grade?|✅ Yes, many companies use this pattern|

---

# 🎯 Final Note

If you expect **extremely frequent changes** (e.g., every second),  
then **broadcast refresh** is NOT the right tool — you should move to **external store** like **Redis + local cache**.

But for **ad click mappings** updating every few minutes — this **dynamic broadcast refresh** method is **perfect** and super clean.

---

Would you also like me to show you:

- An even more **optimized design** where you **cache misses inside tasks** after rebroadcast?
    
- Or a **Scala version** if you're working in JVM Spark?
    

🚀 Tell me!
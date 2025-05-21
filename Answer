# CS653 Final Exam

# ข้อ 1

- **สร้าง S3 bucket:** สร้าง S3 bucket ตั้งชื่อตามรูปแบบที่กำหนด (เช่น `studentxxxx-yyyy-customers`) และสร้างโฟลเดอร์ชื่อ `csv/` ภายใน bucket นั้น.
- **อัปโหลดไฟล์:** อัปโหลดไฟล์ `customers.csv` เข้าไปในโฟลเดอร์ `csv/` ใน S3 bucket.
- **สร้าง Glue Database:** สร้าง Glue Database ใน Glue Data Catalog ตั้งชื่อตามที่กำหนด (เช่น `customers_xxxx_yyyy`).
- **สร้างและกำหนดค่า Glue Crawler:** สร้าง Glue Crawler โดยชี้แหล่งข้อมูลไปที่โฟลเดอร์ `csv/` ใน S3 bucket **เลือกใช้ IAM role ที่มีอยู่ (`labrole`)** และกำหนดให้บันทึก metadata ลงใน Glue Database ที่สร้างไว้.
- **รัน Glue Crawler:** รัน Crawler เพื่อให้ Glue สแกนไฟล์ใน S3 และสร้าง Table Definition ใน Data Catalog.
- **ตรวจสอบ Table:** ตรวจสอบว่ามี Table ถูกสร้างขึ้นใน Glue Database ที่กำหนดใน Glue Data Catalog แล้ว (ควรมีชื่อตาราง เช่น `csv`).
- **ใช้ Athena คิวรีข้อมูล:** ไปที่ Amazon Athena เลือก Database และ Table ที่ Glue สร้างไว้ แล้วเขียนคำสั่ง SQL เพื่อคิวรีข้อมูลลูกค้าที่มาจาก 'Thailand' เช่น `SELECT * FROM "ชื่อฐานข้อมูล"."ชื่อตาราง" WHERE country = 'Thailand';`

# ข้อ 2

- **สร้าง S3 bucket ใหม่และโฟลเดอร์:** สร้าง S3 bucket ใหม่สำหรับข้อมูลสินค้า (เช่น `studentxxxx-yyyy-products`) และสร้างโฟลเดอร์ชื่อ `csv/` ภายใน bucket นั้น.
- **อัปโหลดไฟล์:** อัปโหลดไฟล์ `products.csv` เข้าไปในโฟลเดอร์ `csv/` ใน S3 bucket ใหม่.
- **สร้าง Glue Database ใหม่:** สร้าง Glue Database ใหม่ใน Glue Data Catalog สำหรับข้อมูลสินค้า (เช่น `products_xxxx_yyyy`).
- **ใช้คอนโซล Athena สร้าง Table จาก S3:** ไปที่ Amazon Athena ในคอนโซล และใช้ฟีเจอร์ **"Create"** เลือก **"Create a table from data source"** แล้วเลือก **"S3 bucket data"**. กำหนดค่า **Database** ที่สร้างไว้ ตั้งชื่อ **Table** (เช่น `test`). กำหนด **Location** ชี้ไปที่โฟลเดอร์ `csv/` ใน S3. เลือก **Format** เป็น **CSV**. ในส่วนการกำหนด Schema (Column Details) ให้กำหนดชื่อคอลัมน์และประเภทข้อมูลตามโครงสร้างไฟล์ของคุณ (id bigint, name string, email string, category string, signup_date string).
- **ใช้ Athena คิวรีข้อมูล:** ใน Athena ใช้คำสั่ง SQL คิวรีนี้จากตารางที่คุณสร้างไว้ (เช่น `test`) เพื่อค้นหาและแสดงประเภทสินค้าที่มีผู้ใช้งานสนใจมากที่สุดในปี 2023

```sql
SELECT
    category,
    COUNT(*) AS interested_users_count
FROM
    "products_6120_3987"."test"
WHERE
    TRY_CAST(id AS BIGINT) IS NOT NULL
    AND CAST(substr(signup_date, length(signup_date) - 3) AS INTEGER) = 2023
GROUP BY
    category
ORDER BY
    interested_users_count DESC
LIMIT 1;
```

# ข้อ 3

- **สร้าง SNS Topic และสมัครรับข้อมูล:** สร้าง SNS Topic แบบ **Standard** (ตั้งชื่อตามรูปแบบ เช่น `temperature-alerts-xxxx-yyyy`) และทำการสมัครรับข้อมูลแบบ **Email subscription** โดยใช้ที่อยู่อีเมลของคุณ และ **กดยืนยัน (Confirm)** ในอีเมลที่ได้รับจาก AWS.
- **สร้าง DynamoDB Table:** สร้าง DynamoDB Table (ตั้งชื่อตามรูปแบบ เช่น `iot-temperature-data-xxxx-yyyy`) โดยกำหนด **Partition Key** เป็น **`roomid`** ชนิดข้อมูล **String**.
- **สร้าง Lambda Function:** สร้าง Lambda Function (เลือก Runtime เป็น **Python**) ตั้งชื่อตามรูปแบบ เช่น `iot-temperature-xxxx-yyyy-logs`. เลือกใช้ **`labrole`** เป็น Execution Role. (ทราบว่า Role นี้มีสิทธิ์ครบถ้วนแล้ว).
- **อัปเดต Code ใน Lambda Function:** ไปที่แท็บ **Code** ของ Lambda Function. ลบ Code เดิมทั้งหมดออก แล้ววาง Code นี้ลงไป:
    
    ```sql
    import json
    import boto3
    import os
    from decimal import Decimal
    
    # Initialize AWS clients
    dynamodb = boto3.resource('dynamodb')
    sns = boto3.client('sns')
    
    # --- REPLACE THESE WITH YOUR ACTUAL RESOURCE NAMES/ARNS ---
    # Replace 'arn:aws:sns:us-east-1:304816568247:temperature-alerts-6120-3987'
    # with the ACTUAL ARN of your SNS Topic. Find this in the SNS console.
    sns_topic_arn = 'arn:aws:sns:us-east-1:304816568247:temperature-alerts-6120-3987'
    
    # Replace 'iot-temperature-data-6120-3987'
    # with the ACTUAL name of your DynamoDB Table. Find this in the DynamoDB console.
    dynamodb_table_name = 'iot-temperature-data-6120-3987'
    # --- END OF REPLACEMENT SECTION ---
    
    table = dynamodb.Table(dynamodb_table_name)
    
    def lambda_handler(event, context):
        print("Received event: " + json.dumps(event))
    
        try:
            # Handle cases where the event might be a stringified JSON or a dictionary
            message = event
            if isinstance(event, str):
                 message = json.loads(event)
    
            room_id = message.get('roomid')
            timestamp = message.get('timestamp')
            temperature_raw = message.get('temperature')
    
            if room_id is None or timestamp is None or temperature_raw is None:
                print("Missing required fields in message")
                # Optionally send an SNS alert for invalid data structure
                try:
                    sns.publish(
                        TopicArn=sns_topic_arn,
                        Subject='IoT Temperature Alert - Missing Fields',
                        Message=f'Received message with missing fields: {json.dumps(message)}'
                    )
                except Exception as sns_error:
                     print(f"Failed to send SNS alert for missing fields: {sns_error}")
    
                return {
                    'statusCode': 400,
                    'body': json.dumps('Missing required fields')
                }
    
            # Convert temperature to Decimal for DynamoDB and float for comparison
            try:
                # Convert raw temperature value to string first for robust Decimal conversion
                temperature_decimal = Decimal(str(temperature_raw))
                temperature_float = float(temperature_decimal) # Keep float version for comparison
            except Exception as e: # Catch errors during conversion
                print(f"Invalid temperature value received: {temperature_raw}. Error: {e}")
                # Send SNS alert for invalid data value
                try:
                    sns.publish(
                        TopicArn=sns_topic_arn,
                        Subject='IoT Temperature Alert - Invalid Data Value',
                        Message=f'Received invalid temperature data value: {temperature_raw} for Room ID: {room_id}. Error: {e}'
                    )
                except Exception as sns_error:
                     print(f"Failed to send SNS alert for invalid value: {sns_error}")
    
                return {
                    'statusCode': 400,
                    'body': json.dumps('Invalid temperature value')
                }
    
            # --- Save data to DynamoDB ---
            # Use Decimal type for temperature when putting into DynamoDB
            table.put_item(
                Item={
                    'roomid': str(room_id),
                    'timestamp': str(timestamp),
                    'temperature': temperature_decimal
                }
            )
            print(f"Successfully saved data to DynamoDB for room: {room_id}, temp: {temperature_float}")
    
            # --- Check temperature and send SNS alert if out of range ---
            if temperature_float < 2.0 or temperature_float > 8.0:
                alert_message = f"Temperature out of range! Room ID: {room_id}, Temperature: {temperature_float}°C at {timestamp}"
                print(f"Temperature out of range: {alert_message}")
    
                # Publish alert to SNS
                try:
                    sns.publish(
                        TopicArn=sns_topic_arn,
                        Subject='IoT Temperature Alert - Out of Range',
                        Message=alert_message
                    )
                    print("SNS out-of-range alert sent.")
                except Exception as sns_error:
                     print(f"Failed to send SNS out-of-range alert: {sns_error}")
    
            else:
                print(f"Temperature within range: {temperature_float}°C")
    
            return {
                'statusCode': 200,
                'body': json.dumps('Processing complete')
            }
    
        except Exception as e:
            # Catch any other unexpected errors
            print(f"An unexpected error occurred during event processing: {e}")
            # Send general SNS alert for processing errors
            try:
                sns.publish(
                     TopicArn=sns_topic_arn,
                     Subject='IoT Temperature Alert - Unexpected Lambda Error',
                     Message=f'Unexpected error processing IoT message: {json.dumps(event)}. Error: {e}'
                )
                print("SNS unexpected error alert sent.")
            except Exception as sns_error:
                print(f"Failed to send SNS unexpected error alert: {sns_error}")
    
            return {
                'statusCode': 500,
                'body': json.dumps(f'Error processing event: {e}')
            }
    ```
    
- **ส่วนที่ต้องแก้ไข:** ใน Code ด้านบน ให้แก้ไขค่า **`sns_topic_arn`** และ **`dynamodb_table_name`** ให้เป็น **ARN ของ SNS Topic และชื่อ DynamoDB Table ที่คุณสร้างขึ้นจริงๆ** โดยดูได้จากหน้า console ของบริการนั้นๆ. บรรทัดที่ต้องแก้ไขจะอยู่ใต้ Comment
- **สร้าง IoT Rule:** ไปที่ AWS IoT Core Console > **Message routing** > **Rules**. สร้าง Rule ตั้งชื่อตามรูปแบบ เช่น `IotRule_coldroomxxxx_yyyy`. กำหนด **SQL statement** เป็น `SELECT * FROM 'coldroomxxxx-yyyy/temperature'` (แทนที่ xxxx-yyyy). ในส่วน **Rule actions** เลือก **Send a message to a Lambda function** และเลือก Lambda Function ของคุณ. (สิทธิ์ในการ Invoke Lambda ถูกจัดการโดย Lab Role แล้ว).
- **ทดสอบด้วย MQTT Test Client:**
    - ไปที่ AWS IoT Core > **MQTT test client**.
    - กด **Connect** เพื่อเชื่อมต่อ.
    - ไปที่แท็บ **Subscribe to a topic** และพิมพ์ **`coldroomxxxx-yyyy/temperature`** แล้วกด **Subscribe** เพื่อดูข้อความที่ Publish ไป.
    - ไปที่แท็บ **Publish to a topic**. พิมพ์ **Topic name** เป็น **`coldroomxxxx-yyyy/temperature`**.
    - ในช่อง **Message payload** พิมพ์ข้อมูลอุณหภูมิใน **Format แบบ JSON กระชับ (ไม่มี Newline หรือ Indent เยอะๆ)** เช่น **`{"roomid":"R001","timestamp":"2025-05-07T19:15:00Z","temperature":5.5}`**.
    - ทำการ Publish ข้อความ **10 ครั้ง** โดยมีค่าอุณหภูมิ **อยู่ในช่วง 2°C - 8°C 3 ครั้ง** และ **นอกช่วง 2°C - 8°C  7 ครั้ง**.

# ข้อ 4

- สร้าง Cloud9 Environment ชื่อ `feed-temperature6120-3987` โดยเลือก Instance แบบ `t2.micro`, ระบบปฏิบัติการ Amazon Linux 2023 และเชื่อมต่อแบบ **SSH** แทน SSM (เนื่องจากบัญชี AWS Academy ไม่มีสิทธิ์สร้าง IAM Role เอง) จากนั้นรอให้ Environment สร้างเสร็จและเข้าสู่หน้าจอ Terminal
- สร้าง S3 Bucket ชื่อ `s3-temperature-store-6120-3987` โดยใช้ Region เดียวกับ Cloud9 และตั้งค่าเริ่มต้นทั้งหมดไม่ต้องเปิด public access
- สร้าง Amazon Data Firehose Stream ชื่อ `temperature-stream-6120-3987` โดยตั้งค่า Source เป็น **Direct PUT**, Destination เป็น **Amazon S3**, เลือก Bucket ที่สร้างไว้คือ `s3-temperature-store-6120-3987`, และเลือก IAM Role เป็น `labRole` จากนั้นไม่ต้องเปิดใช้ตัวเลือกเสริมอื่นใด (ไม่เปิด data transformation, format conversion, หรือ decompression)
- กลับไปที่ Cloud9 แล้วสร้างไฟล์ชื่อ `send_temp.py` พร้อมใส่โค้ดต่อไปนี้ลงไป:
    
    ```sql
    import boto3
    import json
    import time
    import random
    
    firehose = boto3.client('firehose', region_name='us-east-1') -- ระวัง Region
    stream_name = 'temperature-stream-6120-3987'
    
    while True:
        temperature_data = {
            'temperature': round(random.uniform(25.0, 40.0), 2),
            'unit': 'C',
            'timestamp': time.strftime('%Y-%m-%d %H:%M:%S')
        }
        response = firehose.put_record(
            DeliveryStreamName=stream_name,
            Record={
                'Data': json.dumps(temperature_data) + '\n'
            }
        )
        print(f"Sent: {temperature_data}")
        time.sleep(3)
    
    ```
    

หลังจากบันทึกไฟล์แล้วให้รันคำสั่ง `python3 send_temp.py` ใน Terminal แล้วปล่อยให้รันอย่างน้อย 5 นาทีเพื่อรอให้ Firehose ทำการ buffer ข้อมูลและส่งเข้า S3

- ตรวจสอบ S3 Bucket ที่ `s3-temperature-store-6120-3987` จะพบไฟล์ `.json.gz` ปรากฏภายใน path ที่สร้างโดย Firehose โดยอัตโนมัติตามวันและเวลา เช่น `/2025/05/08/06/` ขนาดไฟล์ประมาณ 7KB ซึ่งภายในจะเป็นข้อมูล JSON ที่ส่งมาจาก Python script

# ข้อ 5

- อัปโหลดไฟล์ `feedbacks.csv` ไปยัง S3 ที่ path: `s3://student6120-3987-ml/raw/`
    - สร้าง S3 Bucket
    - สร้าง Folder ชื่อ raw
- สร้าง Glue Job > Visual ETL > กรอกข้อมูลต่างๆในแถบ Job Details และใส่ Code ในแถบ Script

| รายการ | คำอธิบาย |
| --- | --- |
| **Job name** | `csv_to_json_preprocess` หรือชื่อที่สื่อความหมาย |
| **IAM Role** | `AWSGlueServiceRoleDefault` หรือ `LabRole` (ต้องมีสิทธิ์เขียน S3) |

```python
import sys
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

df = spark.read.option("header", "true").csv("s3://student6120-3987-ml/raw/feedbacks.csv")
df.write.mode("overwrite").json("s3://student6120-3987-ml/preprocessed/")

job.commit()
```

- สร้าง Sentiment Endpoint ด้วย CloudFormation
    - Deploy Template ด้วย `sentiment.yaml` ที่ให้ไว้
    - ตั้งชื่อ Stack ตามต้องการ เช่น `sentiment-api`
    - เลือก VPC
    - เลือก LabRole ถ้ามี
    - เมื่อสร้างเสร็จ → ไปที่แท็บ `Outputs` → คัดลอกค่า `SentimentEndpoint` ที่ลงท้ายด้วย `/predict`
    ตัวอย่าง endpoint > [`http://ec2-44-201-210-179.compute-1.amazonaws.com/predict`](http://ec2-44-201-210-179.compute-1.amazonaws.com/predict)
- แก้โค้ดใน Glue Job > save > run

```python
import sys
import json
import requests
import pandas as pd
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job
from pyspark.sql.functions import pandas_udf
from pyspark.sql.types import DoubleType, StringType

# 🔧 Replace with your deployed endpoint
SENTIMENT_ENDPOINT = "http://ec2-44-201-210-179.compute-1.amazonaws.com/predict" --อย่าลืมเปลี่ยนอันนี้

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# ✅ Load preprocessed JSON (not CSV!) from S3
input_df = spark.read.json("s3://student6120-3987-ml/preprocessed/")

# ✅ Define sentiment analysis function with error key always included
def analyze_sentiment(row):
    base = {
        "text": row['text'],
        "sentiment": None,
        "score_positive": None,
        "score_negative": None,
        "score_neutral": None,
        "error": None
    }
    try:
        response = requests.post(
            SENTIMENT_ENDPOINT,
            json={"text": row['text']},
            timeout=5
        )
        result = response.json()
        base.update({
            "sentiment": result.get("sentiment"),
            "score_positive": result.get("scores", {}).get("positive"),
            "score_negative": result.get("scores", {}).get("negative"),
            "score_neutral": result.get("scores", {}).get("neutral")
        })
    except Exception as e:
        base["sentiment"] = "ERROR"
        base["error"] = str(e)
    return base

# ✅ UDF wrapper
@pandas_udf("text string, sentiment string, score_positive double, score_negative double, score_neutral double, error string")
def predict_udf(text_series: pd.Series) -> pd.DataFrame:
    results = []
    for text in text_series:
        row = {"text": text}
        results.append(analyze_sentiment(row))
    return pd.DataFrame(results)

# ✅ Process and write output
output_df = input_df.select("text")
final_df = output_df.withColumn("result", predict_udf("text")).selectExpr("result.*")
final_df.write.mode("overwrite").json("s3://student6120-3987-ml/processed/")

job.commit()

```

- เข้า Athena แล้ว Run SQL 2 อันด้านล่างตามลำดับ

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS sentiment_results (
  text STRING,
  sentiment STRING,
  score_positive DOUBLE,
  score_negative DOUBLE,
  score_neutral DOUBLE,
  error STRING
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://student6120-3987-ml/processed/';
```

```sql
SELECT sentiment, COUNT(*) AS total
FROM sentiment_results
WHERE sentiment IS NOT NULL
GROUP BY sentiment;
```

# ข้อ  6

- สร้าง Bucket `s3://studentxxxx-yyyy-ml/` > สร้าง Folder raw
- แตกไฟล์และอัปโหลดภาพไปยัง S3 ที่ path: `s3://studentxxxx-yyyy-ml/raw/`
- สร้าง Glue Job > Visual ETL > กรอกข้อมูลต่างๆในแถบ Job Details และใส่ Code ในแถบ Script > run

```python
import sys
import boto3
import json
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# กำหนด bucket และ prefix ที่เก็บภาพ
bucket = "student6120-3987-ml"
prefix = "raw/"

# ดึงรายการไฟล์ใน S3
s3 = boto3.client('s3')
response = s3.list_objects_v2(Bucket=bucket, Prefix=prefix)

# สร้าง JSON ที่มีทั้งชื่อและ URI
files = [
    {
        "filename": obj["Key"].split("/")[-1],
        "s3uri": f"s3://{bucket}/{obj['Key']}"
    }
    for obj in response.get("Contents", [])
    if obj["Key"].endswith(".jpg")
]

# เขียนข้อมูล JSON ลง S3
rdd = sc.parallelize(files)
df = spark.read.json(rdd)
df.write.mode("overwrite").json("s3://student6120-3987-ml/preprocessed/")

job.commit()

```

- เปลี่ยน code Glue Job เป็นด้านล่าง > save > run

```python
import sys
import json
import boto3
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job
from pyspark.sql.types import StructType, StructField, StringType, BooleanType, DoubleType

# Step 1: Glue job setup
args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# Step 2: Load input JSON from preprocessed path
input_df = spark.read.json("s3://student6120-3987-ml/preprocessed/")

# Step 3: Define output schema explicitly
schema = StructType([
    StructField("filename", StringType(), True),
    StructField("has_face", BooleanType(), True),
    StructField("gender", StringType(), True),
    StructField("confidence", DoubleType(), True),
    StructField("error", StringType(), True)
])

# Step 4: Detect faces using AWS Rekognition
def detect_face(row):
    result = {
        "filename": row["filename"],
        "has_face": False,
        "gender": None,
        "confidence": None,
        "error": None
    }
    try:
        rekognition = boto3.client('rekognition')
        response = rekognition.detect_faces(
            Image={"S3Object": {
                "Bucket": "student6120-3987-ml",
                "Name": f"raw/{row['filename']}"
            }},
            Attributes=["ALL"]
        )
        if response["FaceDetails"]:
            face = response["FaceDetails"][0]
            result["has_face"] = True
            result["gender"] = face["Gender"]["Value"]
            result["confidence"] = face["Gender"]["Confidence"]
    except Exception as e:
        result["error"] = f"{str(e)} | file={row['filename']}"  # ✅ more traceable

    return result

# Step 5: Apply and write results to processed/
rows_rdd = input_df.rdd.map(lambda row: detect_face(row.asDict()))
output_df = spark.createDataFrame(rows_rdd, schema=schema)
output_df.write.mode("overwrite").json("s3://student6120-3987-ml/processed/")

job.commit()

```

- สร้าง Table ใน AWS Athena

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS processed_faces (
  filename   string,
  has_face   boolean,
  gender     string,
  confidence double,
  error      string
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://student6120-3987-ml/processed/'
TBLPROPERTIES ('has_encrypted_data'='false');

```

- จำนวนภาพที่ตรวจพบใบหน้า

```sql
SELECT COUNT(*) AS face_detected_count
FROM processed_faces
WHERE has_face = true;
```

- จำนวนเพศที่ตรวจพบ (เฉพาะภาพที่มีใบหน้า)

```sql
SELECT gender, COUNT(*) AS count
FROM processed_faces
WHERE has_face = true
GROUP BY gender;
```

- จำนวนภาพที่ไม่พบใบหน้า

```sql
SELECT COUNT(*) AS no_face_count
FROM processed_faces
WHERE has_face = false;
```

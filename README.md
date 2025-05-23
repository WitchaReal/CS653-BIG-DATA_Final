## 🧩 ข้อที่ 1: Upload + Glue Crawler + Athena
## ✅ ขั้นตอนที่ 1: Upload customers.csv ไปยัง S3

1. เข้า AWS Console → S3 → \[Create bucket]

2. ตั้งชื่อ bucket:

   ```
   student6196-YYYY-customers
   ```

3. เปิด bucket → กด \[Create folder] → ตั้งชื่อ: `csv`

4. เข้าไปในโฟลเดอร์ `csv` → กด \[Upload] → เลือกไฟล์ `customers.csv` → กด \[Upload]

---

## ✅ ขั้นตอนที่ 2: สร้าง Glue Database

1. เปิด AWS Console → Glue → กดเมนู `Databases` (ในหัวข้อ Data Catalog)
2. กด \[Add database] → ตั้งชื่อว่า:

   ```
   customers_6196_YYYY
   ```

   (เช่น customers\_6196\_1234) → \[Create database]

---

## ✅ ขั้นตอนที่ 3: สร้าง Glue Crawler

1. ไปที่ Glue → เลือก `Crawlers` → \[Create crawler]

2. ตั้งชื่อ:

   ```
   crawler-customers-6196-YYYY
   ```

3. Data source:

   * Data store: S3
   * Include path: `s3://student6196-YYYY-customers/csv/`

4. IAM role: เลือก `LabRole` หรือ role ที่มีสิทธิ

5. Output:

   * Database: `customers_6196_YYYY`
   * Table prefix: `customers_` (ใส่หรือไม่ใส่ก็ได้)

6. สร้างเสร็จ → กด Run → รอจน Crawler เป็น “**Ready**”

---

## ✅ ขั้นตอนที่ 4: ใช้ Athena Query ข้อมูล

1. เปิด Athena → ตรวจสอบ Region ให้ตรงกับ S3 และ Glue

2. ตั้งค่า Query results location (ครั้งแรกเท่านั้น):

   * กด Settings → Manage → เลือก S3 เช่น:

     ```
     s3://student6196-YYYY-customers/athena-results/
     ```

3. เลือก Database: `customers_6196_YYYY` (เมนูซ้าย)

4. คลิก \[New Query] → พิมพ์ SQL ด้านล่างนี้:

```sql
SELECT * 
FROM customers.csv
WHERE country = 'Thailand';
```

> เปลี่ยน `customers_xxxx_yyyy` เป็นชื่อตารางจริง เช่น `customers_6196_1234`

5. กด \[Run] แล้วรอผลลัพธ์

---

## ✅ ขั้นตอนที่ 5: Capture หน้าจอ

📸 ภาพเดียวต้องมี:

* Glue Crawler ที่สถานะเป็น “**Ready**”
* ผลลัพธ์จาก Athena ที่แสดงข้อมูลลูกค้าในประเทศ `Thailand`

----
=
=
=
=
=
=
----


## 🧩 ข้อที่ 2: สร้าง Table ด้วย SQL และ Query หาหมวดหมู่ยอดนิยมในปี 2023
## ✅ Step 1: อัปโหลด `products.csv` ไปยัง S3

1. ไปที่ AWS Console → เข้าเมนู S3
2. กด \[Create bucket] → ตั้งชื่อเช่น:

```
student6196-YYYY-products
```

> เช่น `student6196-yyyy-products`

3. เปิด bucket → \[Create folder] → ชื่อว่า: `csv`

4. เข้าไปในโฟลเดอร์ `csv` → กด \[Upload] → เลือกไฟล์ `products.csv` → \[Upload] รอจน 100%

---

## ✅ Step 2: สร้าง Glue Database

1. เข้า AWS Console → ไปที่ Glue
2. เมนูซ้าย → กด `Databases` → \[Add database]
3. ชื่อว่า:

```
products_6196_YYYY
```

> เช่น `products_6196_1234`

4. กด \[Create database]

---

## ✅ Step 3: ใช้ Athena สร้าง Table ด้วย SQL

1. ไปที่ AWS Athena → ตรวจสอบว่า Region ตรงกับ S3/Glue

2. ตั้งค่า Query Result Location (ถ้ายังไม่เคยตั้ง)

   * กด Settings → Manage → ใส่ S3 เช่น:

     ```
     s3://student6196-YYYY-products/athena-results/
     ```

3. เมนูซ้าย: เลือก Database → `products_6196_YYYY`

4. กด \[+ New Query] แล้วใช้ SQL ด้านล่างนี้เพื่อ **สร้าง Table**

### 📄 ตัวอย่าง SQL สร้าง Table:

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS products_6196_yyyy_products (
  user_id STRING,
  product_id STRING,
  category STRING,
  signup_date STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
  'separatorChar' = ',',
  'quoteChar' = '\"'
)
LOCATION 's3://student6196-yyyy-products/csv/'
TBLPROPERTIES ('has_encrypted_data'='false');
```

> เปลี่ยน:
> * `LOCATION` ให้ตรงกับ path ของ S3

5. กด \[Run] → ถ้าสร้างสำเร็จ ตารางจะโผล่ทางด้านซ้าย

---

## ✅ Step 4: ตรวจสอบข้อมูลใน Table

รัน SQL ตรวจดูตัวอย่างข้อมูล:

```sql
SELECT * FROM products_6196_yyyy_products
LIMIT 10;
```

> เช็คให้แน่ใจว่า `event_time` มีข้อมูลปี 2023 อยู่

---

## ✅ Step 5: Query หา Top 1 category ที่มีคนสนใจมากสุดในปี 2023

```sql
SELECT category, COUNT(*) AS interested_count
FROM products_6196_yyyy_products
WHERE regexp_like(signup_date, '.*2023$')
# event_time LIKE '2023%'
GROUP BY category
ORDER BY interested_count DESC
LIMIT 1;
```

---

## ✅ Step 6: Capture ภาพส่ง

📸 **Capture 1 ภาพ ต้องเห็นทั้งหมดนี้**:

1. SQL ที่ใช้สร้าง Table (Query ด้านบน)
2. Preview ข้อมูลใน Table (`SELECT *`)
3. ผลของ Query ที่แสดง category ยอดนิยมปี 2023
----
=
=
=
=
=
=
----

## 🧩 ข้อที่ 3: ระบบติดตามอุณหภูมิห้องเย็นด้วย IoT + Lambda + DynamoDB + SNS

---
### ✅ Step 1: สร้าง DynamoDB Table

1. เข้า AWS Console → ค้นหา "DynamoDB"
2. กด \[Create Table]
3. กำหนดค่าดังนี้:

   * Table name: `iot-temperature-data-6196-YYYY`
   * Partition key: `roomId` (String)
   * Sort key: `timestamp` (String) *(เพิ่มได้จะช่วยจัดเรียงเวลา)*

✅ เสร็จแล้วจะใช้เก็บข้อมูลอุณหภูมิ

---

### ✅ Step 2: สร้าง SNS Topic และ Subscribe Email

1. เข้า AWS Console → ค้นหา "SNS"

2. ไปที่ Topics → กด \[Create topic]

3. ตั้งค่า:

   * Type: Standard
   * Name: `temperature-alerts-6196-YYYY`

4. เปิด Topic → กด \[Create subscription]

   * Protocol: Email
   * Endpoint: ใส่อีเมลของตัวเอง → กด \[Create subscription]

5. ไปยืนยันอีเมลที่ได้รับ (กด "Confirm subscription")

---

### ✅ Step 3: สร้าง Lambda Function

1. เข้า AWS Console → Lambda → \[Create function]
2. เลือก:

   * Name: `iot-temperature-6196-YYYY-logs`
   * Runtime: Python 3.9
   * Permissions: ใช้ `LabRole` หรือ role ที่มีสิทธิ์เขียน DynamoDB และส่ง SNS

---

### ✅ โค้ด Python สำหรับ Lambda:

```python
import json
import boto3
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')

# ใส่ชื่อ Table และ SNS ARN ของคุณ
table_name = 'iot-temperature-data-6196-YYYY'
sns_topic_arn = 'arn:aws:sns:region:account-id:temperature-alerts-XXXX-YYYY'

table = dynamodb.Table(table_name)

def lambda_handler(event, context):
    for record in event['records']:
        payload = json.loads(record['value'])

        room_id = payload['roomId']
        timestamp = payload['timestamp']
        temperature = Decimal(str(payload['temperature']))

        # บันทึกลง DynamoDB
        table.put_item(
            Item={
                'roomId': room_id,
                'timestamp': timestamp,
                'temperature': temperature
            }
        )

        # ตรวจสอบช่วงอุณหภูมิ
        if temperature < 2 or temperature > 8:
            message = f"Alert: Room {room_id} temperature is {temperature}°C at {timestamp}."
            sns.publish(
                TopicArn=sns_topic_arn,
                Message=message,
                Subject="Cold Room Temperature Alert"
            )

    return {
        'statusCode': 200,
        'body': json.dumps('Processed successfully')
    }
```

> 📌 **อย่าลืมเปลี่ยน:**
>
> * `table_name`
> * `sns_topic_arn` ให้ตรงกับของคุณเอง

---

### ✅ Step 4: สร้าง IoT Rule

1. เข้า AWS Console → AWS IoT Core → Message Routing → \[Create rule]

2. กำหนด:

   * Name: `IoTRule_coldroom6196_YYYY`
   * SQL statement:

     ```sql
     SELECT * FROM 'coldroom6196-YYYY/temperature'
     ```
   * SQL version: 2016-03-23

3. Add Action → เลือก: **Invoke Lambda function**

   * เลือก `iot-temperature-6196-YYYY-logs`
   * Allow permissions

---

### ✅ Step 5: ทดสอบส่ง MQTT Messages

1. เข้า AWS IoT Core → MQTT Test Client
2. กด "Publish to a topic"

#### 📌 Topic:

```
coldroom6196-YYYY/temperature
```

#### 📌 ตัวอย่าง Payload (อุณหภูมิในช่วง):

```json
{
  "roomId": "R001",
  "timestamp": "2025-05-21T10:15:00Z",
  "temperature": 5.5
}
```

#### 📌 ตัวอย่าง Payload (อุณหภูมิผิดปกติ):

```json
{
  "roomId": "R002",
  "timestamp": "2025-05-21T10:20:00Z",
  "temperature": 10.2
}
```

> ✳️ ส่งทั้งหมด **10 ครั้ง**:
>
> * 3 ครั้ง: อยู่ในช่วง 2–8°C
> * 7 ครั้ง: นอกช่วง

---
### ✅ Step 6: ตรวจสอบผลลัพธ์

* **DynamoDB**: เข้า Table → ดู Items → ต้องเห็นข้อมูลถูกบันทึก
* **SNS (Email)**: จะได้รับอีเมลแจ้งเตือนทุกครั้งที่อุณหภูมินอกช่วง
---
### ✅ Step 7: Capture ภาพเดียว

📸 ภาพ Capture ที่ต้องส่ง:

* IoT Rule + SQL Statement + Lambda ที่เชื่อมต่อ
* โค้ด Lambda บนหน้าจอ
* DynamoDB ที่แสดงข้อมูลที่ถูกบันทึก
* ภาพหน้าจออีเมลที่ได้รับการแจ้งเตือน (จาก SNS)
----
=
=
=
=
=
=
----

## 🧩 ข้อที่ 4: ระบบส่งอุณหภูมิแบบสุ่มผ่าน Cloud9 → Firehose → S3

---

### ✅ Step 1: สร้าง S3 Bucket เพื่อเก็บข้อมูล

1. ไปที่ AWS Console → S3 → กด \[Create bucket]

2. ตั้งชื่อ Bucket เช่น:

   ```
   s3-temperature-store-6196-YYYY
   ```

   (เปลี่ยน `XXXX-YYYY` ให้ตรงกับรหัสของนักศึกษา)

3. Region: เลือกให้ตรงกับที่คุณจะใช้กับ Cloud9 และ Firehose (เช่น `us-east-1`)

4. กด \[Create bucket]

---

### ✅ Step 2: สร้าง Amazon Kinesis Data Firehose

1. AWS Console → ไปที่ **Amazon Data Firehose**
2. กด \[Create delivery stream]

#### ตั้งค่าดังนี้:

* **Source**: Direct PUT or other sources

* **Destination**: Amazon S3

* **Delivery stream name**:

  ```
  temperature-stream-6196-YYYY
  ```

* **S3 Bucket**: เลือก `s3-temperature-store-6196-YYYY`

* **S3 prefix (optional)**: `temperature-data/`

* **Permissions**: เลือก `LabRole` หรือ Role ที่มีสิทธิ

3. กด \[Create delivery stream]

---

### ✅ Step 3: สร้าง AWS Cloud9 Environment

1. ไปที่ AWS Console → Cloud9 → กด \[Create environment]

#### ตั้งค่า:

* Name: `feed_temperature6196_YYYY`
* Instance type: `t2.micro`
* Platform: `Amazon Linux 2`
* Cost-saving: ค่า default ได้เลย
* Permissions: ใช้ Role เดิม (LabRole)

2. กด \[Create environment]

---

### ✅ Step 4: เขียน Python Script เพื่อส่งอุณหภูมิ → Firehose

#### 🔧 ติดตั้ง Boto3 (ถ้ายังไม่มี):

เปิด Terminal:

```bash
pip install boto3
```

#### ✏️ สร้างไฟล์ใหม่ชื่อ `send_temperature.py` แล้วใส่โค้ดนี้:

```python
import boto3
import json
import time
import random
from datetime import datetime

# กำหนดชื่อ Firehose stream และ region
delivery_stream_name = 'temperature-stream-6196-YYYY'
region_name = 'us-east-1'  # เปลี่ยนให้ตรงกับ Region ของคุณ

client = boto3.client('firehose', region_name=region_name)

def generate_temperature_data():
    return {
        "roomId": f"R{random.randint(1, 5):03d}",
        "timestamp": datetime.utcnow().isoformat(),
        "temperature": round(random.uniform(-5, 15), 2)
    }

while True:
    data = generate_temperature_data()
    json_data = json.dumps(data)
    print("Sending:", json_data)

    response = client.put_record(
        DeliveryStreamName=delivery_stream_name,
        Record={"Data": json_data + "\n"}
    )

    print("Response:", response['ResponseMetadata']['HTTPStatusCode'])
    time.sleep(3)
```

> ✅ เปลี่ยน `delivery_stream_name` และ `region_name` ให้ตรงกับของคุณ

---

### ✅ Step 5: รันโปรแกรมใน Cloud9

```bash
python send_temperature.py
```

📌 Terminal จะแสดงว่า:

* ข้อมูลถูกสุ่มและส่งทุก 3 วินาที
* บอกสถานะ Response: 200 (สำเร็จ)

---

### ✅ Step 6: ตรวจสอบไฟล์ใน S3

1. ไปที่ S3 → เปิด bucket `s3-temperature-store-6196-YYYY`
2. เข้าโฟลเดอร์ `temperature-data/` → จะเห็นไฟล์ `.json` หรือ `.gz` (เป็นการบีบอัดอัตโนมัติ)
3. เปิดไฟล์ → ข้อมูลแต่ละบรรทัดคือ JSON 1 รายการ เช่น:

```json
{"roomId": "R001", "timestamp": "2025-05-21T10:50:00Z", "temperature": 7.82}
```

---

## ✅ Step 7: Capture ภาพเดียว

📸 ภาพ Capture ที่ต้องส่ง:

* Firehose Console → แสดงชื่อ `temperature-stream-6196XXXX-YYYY`
* Cloud9 → เปิดไฟล์ `send_temperature.py` ให้เห็นโค้ด
* S3 Console → เปิดดู bucket + แสดงไฟล์ที่เก็บได้จริง
----
=
=
=
=
=
=
----

## 🧩 ข้อที่ 5: วิเคราะห์ Sentiment จาก Feedback โดยใช้ Glue + AI Model
---

### ✅ Step 1: อัปโหลดไฟล์ `feedbacks.csv` ไป S3

1. ไปที่ AWS Console → S3

2. สร้าง Bucket ชื่อ:

   ```
   student6196-YYYY-ml
   ```

3. ภายใน Bucket → สร้างโฟลเดอร์ชื่อ:

   ```
   raw/
   ```

4. อัปโหลดไฟล์ `feedbacks.csv` ลงใน `raw/`

--- 

### ✅ Step 2: Glue Job  แปลง CSV → JSON

1. ไปที่ AWS Glue Studio → Create job → *Visual with a source and target*

2. ตั้งชื่อ: `csv-to-json-job`

3. Source: Amazon S3 → Browse ไปที่:

   ```
   s3://student6196-YYYY-ml/raw/feedbacks.csv
   ```

4. Format: CSV → Set header = true

5. Target: Amazon S3 → Location:

   ```
   s3://student6196-YYYY-ml/preprocessed/
   ```

6. Format: JSON

7. กด **Run Job**

8. ตรวจสอบว่าไฟล์ JSON ถูกสร้างใน `preprocessed/`

---

### ✅ Step 3: Deploy Sentiment API ด้วย CloudFormation

1. ไปที่ AWS Console → CloudFormation
2. \[Create Stack] → With new resources
3. Upload ไฟล์ `sentiment-api.yaml`
4. Stack name: `sentiment-analysis-stack`
5. กด Next → Next → Create stack

📌 รอจน `Status = CREATE_COMPLETE`

6. ไปที่แท็บ **Outputs** → เก็บค่า `SentimentEndpoint` (URL เช่น: `http://ec2-xx-xx-xx-xx.compute-1.amazonaws.com/predict`)

---

### ✅ Step 4: ทดสอบ API ด้วย Web Browser

เปิด Browser แล้วเข้า:

```
http://ec2-xx-xx-xx-xx.compute-1.amazonaws.com/predict
```

หากพร้อมใช้งาน → จะขึ้น:

```json
{"detail":"Method Not Allowed"}
```

📸 *Capture ภาพหน้านี้ไว้ด้วย*

---

### ✅ Step 5: Glue Python Shell Job วิเคราะห์ Sentiment

1. ไปที่ AWS Glue → Jobs → \[Create job]
2. เลือก: *Author script with Python Shell*
3. ตั้งชื่อ: `sentiment-analysis-job`
4. IAM Role: LabRole
5. Python version: 3.9

---

### 🧠 ตัวอย่างโค้ด Glue Python Shell:

```python
import json
import boto3
import requests

s3 = boto3.client('s3')

# เปลี่ยน `input_bucket`, `endpoint`, `input_key`, และ `output_key` ให้ตรงกับของคุณ
input_bucket = 'student6196-YYYY-ml'
input_key = 'preprocessed/feedbacks.json'
output_key = 'processed/sentiment_results.json'
endpoint = 'http://ec2-xx-xx-xx-xx.compute-1.amazonaws.com/predict'

# ดึงข้อมูล feedback จาก S3
obj = s3.get_object(Bucket=input_bucket, Key=input_key)
feedbacks = json.loads(obj['Body'].read())

results = []

for feedback in feedbacks:
    text = feedback['text']
    res = requests.post(endpoint, json={'text': text})
    sentiment_data = res.json()
    
    results.append({
        'id': feedback['id'],
        'text': text,
        'sentiment': sentiment_data['sentiment'],
        'scores': sentiment_data['scores']
    })

# เขียนผลลัพธ์ลง S3
s3.put_object(
    Body=json.dumps(results, indent=2),
    Bucket=input_bucket,
    Key=output_key
)
```

> ✴️ เปลี่ยน `input_bucket`, `endpoint`, `input_key`, และ `output_key` ให้ตรงกับของคุณ

---

### ✅ Step 6: ตรวจสอบใน S3 → `processed/sentiment_results.json`

เปิดดูว่าแต่ละ feedback ถูกวิเคราะห์ออกมาแบบนี้:

```json
{
  "id": 3,
  "text": "I absolutely love this product!",
  "sentiment": "POSITIVE",
  "scores": {
    "positive": 0.95,
    "negative": 0.02,
    "neutral": 0.03
  }
}
```

---

### ✅ Step 7: ใช้ Athena Query สรุป Sentiment

#### 📌 สร้าง Table:

```sql
CREATE EXTERNAL TABLE sentiment_results (
  id INT,
  text STRING,
  sentiment STRING,
  scores STRUCT<positive: DOUBLE, negative: DOUBLE, neutral: DOUBLE, mixed: DOUBLE>
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://studentXXXX-YYYY-ml/processed/';
```

#### 📌 Query หาค่าที่เจอบ่อยที่สุด:

```sql
SELECT sentiment, COUNT(*) AS count
FROM sentiment_results
GROUP BY sentiment
ORDER BY count DESC
LIMIT 1;
```

---

## ✅ ภาพ Capture ที่ต้องส่ง

📸 *รวมภาพทั้งหมดไว้ในภาพเดียวหรือ collage*

1. หน้าจอ Glue Job ที่เห็น pipeline หรือ script ทั้งหมด
2. หน้า browser แสดง `{"detail":"Method Not Allowed"}`
3. หน้า S3 ที่มี `sentiment_results.json`
4. หน้าจอ Athena แสดงผลสรุป sentiment

----
=
=
=
=
=
=
----
## 🧩 ข้อที่ 6: วิเคราะห์ภาพใบหน้าด้วย AWS Rekognition + Glue
---

### 🎯 เป้าหมายของโจทย์:
1. อัปโหลดไฟล์ภาพ 20 รูปไปยัง S3
2. สร้าง Glue Job:
   * สร้างไฟล์ JSON ที่เก็บชื่อไฟล์และ S3 URI
   * ใช้ `Rekognition` ตรวจจับใบหน้า (เพศ + ความมั่นใจ)
   * บันทึกผลลัพธ์ JSON ลง S3
3. ใช้ Athena:
   * นับภาพที่พบใบหน้า
   * นับจำนวนเพศ
   * นับภาพที่ไม่พบใบหน้า
---

### ✅ Step 1: อัปโหลดรูปภาพไปยัง S3

1. สร้าง S3 bucket ชื่อ:

   ```
   student6196-YYYY-ml
   ```

2. สร้างโฟลเดอร์ `raw/`

3. อัปโหลดไฟล์ `1.jpg` ถึง `20.jpg` ทั้งหมดไปยัง:

   ```
   s3://student6196-YYYY-ml/raw/
   ```

---

### ✅ Step 2: Glue Job > Visual ETL > กรอกข้อมูลต่างๆในแถบ Job Details และใส่ Code ในแถบ Script > run  > Python Shell Job ที่ 1 → สร้าง image\_list.json

> Job นี้อ่านชื่อไฟล์ใน S3 → สร้างไฟล์ JSON พร้อม S3 URI ของแต่ละไฟล์

#### 🔧 ตัวอย่างโค้ด:

```python
import boto3
import json

bucket = 'student6196-YYYY-ml'
prefix = 'raw/'

s3_client = boto3.client('s3')
response = s3_client.list_objects_v2(Bucket=bucket, Prefix=prefix)

images = []
for obj in response.get('Contents', []):
    key = obj['Key']
    if key.endswith('.jpg'):
        images.append({
            'filename': key.split('/')[-1],
            's3_uri': f's3://{bucket}/{key}'
        })

output_data = {'images': images}

s3_client.put_object(
    Bucket=bucket,
    Key='preprocessed/image_list.json',
    Body=json.dumps(output_data, indent=2)
)
```

---

### ✅ Step 3: Glue Python Shell Job ที่ 2 → วิเคราะห์ใบหน้าด้วย Rekognition

> อ่าน `image_list.json` → ใช้ `detect_faces()` → สร้างผลลัพธ์ JSON ทีละไฟล์

#### 🔧 ตัวอย่างโค้ด:

```python
import boto3
import json
import urllib.parse

bucket = 'student6196-YYYY-ml'
rekognition = boto3.client('rekognition')
s3 = boto3.client('s3')

# ดึง image_list.json
obj = s3.get_object(Bucket=bucket, Key='preprocessed/image_list.json')
data = json.loads(obj['Body'].read())
images = data['images']

for image in images:
    filename = image['filename']
    s3_uri = image['s3_uri']
    s3_key = '/'.join(s3_uri.split('/', 4)[-1].split('/'))

    response = rekognition.detect_faces(
        Image={'S3Object': {'Bucket': bucket, 'Name': s3_key}},
        Attributes=['ALL']
    )

    if response['FaceDetails']:
        face = response['FaceDetails'][0]
        gender = face['Gender']['Value']
        confidence = face['Gender']['Confidence']
        has_face = True
    else:
        gender = None
        confidence = None
        has_face = False

    result = {
        'filename': filename,
        'has_face': has_face,
        'gender': gender,
        'confidence': confidence
    }

    s3.put_object(
        Bucket=bucket,
        Key=f'processed/{filename.replace(".jpg", ".json")}',
        Body=json.dumps(result, indent=2)
    )
```

---

### ✅ Step 4: ตรวจสอบไฟล์ JSON ใน S3

1. ไปที่ S3:

   * `preprocessed/image_list.json` → รายชื่อภาพทั้งหมด
   * `processed/` → JSON 20 ไฟล์ (ชื่อเช่น `1.json`, `2.json`, ...)

📄 ตัวอย่างไฟล์ผลลัพธ์:

```json
{
  "filename": "7.jpg",
  "has_face": true,
  "gender": "Male",
  "confidence": 99.3
}
```

---

### ✅ Step 5: สร้าง Table ใน Athena

```sql
CREATE EXTERNAL TABLE face_results (
  filename STRING,
  has_face BOOLEAN,
  gender STRING,
  confidence DOUBLE
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://studentXXXX-YYYY-ml/processed/';
```

---

### ✅ Step 6: Athena Queries

#### 🔹 นับจำนวนภาพที่พบใบหน้า

```sql
SELECT COUNT(*) AS face_detected
FROM face_results
WHERE has_face = true;
```

#### 🔹 นับจำนวนเพศที่พบ

```sql
SELECT gender, COUNT(*) AS count
FROM face_results
WHERE has_face = true
GROUP BY gender;
```

#### 🔹 นับจำนวนภาพที่ไม่พบใบหน้า

```sql
SELECT COUNT(*) AS no_face_count
FROM face_results
WHERE has_face = false;
```

---

### ✅ Step 7: Capture ภาพส่ง

📸 ต้องรวมสิ่งเหล่านี้ในภาพเดียว:

* Glue Job ที่ใช้วิเคราะห์ (ถ้าใช้ script → แสดงโค้ด)
* โฟลเดอร์ `preprocessed/` → เปิดดู `image_list.json`
* โฟลเดอร์ `processed/` → เปิดดู 1 ไฟล์ JSON
* ผลของ Athena Query:

  * ภาพที่พบใบหน้า
  * เพศที่พบ
  * ภาพที่ไม่พบใบหน้า

---






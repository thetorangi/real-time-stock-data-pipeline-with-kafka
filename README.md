# 📈 Real-Time Stock Data Pipeline with Apache Kafka

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![Kafka](https://img.shields.io/badge/Apache%20Kafka-2.8+-red.svg)](https://kafka.apache.org/)
[![AWS](https://img.shields.io/badge/AWS-S3%20%7C%20Glue%20%7C%20Athena-orange.svg)](https://aws.amazon.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> **An end-to-end real-time data engineering solution for processing financial market data using Apache Kafka, Python, and AWS services.**

---

## 🎯 Project Overview

Financial markets generate massive amounts of data every second. This project demonstrates a **production-ready data pipeline** that ingests, processes, stores, and queries real-time stock/index data efficiently.

### Key Features

- ⚡ **Real-time data ingestion** using Apache Kafka
- 🔄 **Stream processing** with Python consumers
- ☁️ **Scalable cloud storage** on AWS S3
- 📊 **Automated schema management** via AWS Glue
- 🔍 **SQL-based analytics** using AWS Athena
- 🛡️ **Fault-tolerant** and horizontally scalable architecture

---

## 🏗️ Architecture

```
┌─────────────────┐      ┌──────────────┐      ┌─────────────┐
│  Data Source    │─────▶│    Kafka     │─────▶│   Python    │
│  (API/CSV)      │      │   Producer   │      │  Consumer   │
└─────────────────┘      └──────────────┘      └──────┬──────┘
                                                       │
                                                       ▼
                         ┌─────────────────────────────────┐
                         │         AWS S3 (Data Lake)      │
                         │    Partitioned by Index/Date    │
                         └────────────┬────────────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    ▼                 ▼                 ▼
            ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
            │  AWS Glue    │  │  AWS Glue    │  │  AWS Athena  │
            │   Crawler    │─▶│   Catalog    │◀─│   (Queries)  │
            └──────────────┘  └──────────────┘  └──────────────┘
```

---

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [Dataset Schema](#-dataset-schema)
- [Pipeline Workflow](#-pipeline-workflow)
- [AWS Setup](#-aws-setup)
- [SQL Queries](#-sql-queries)
- [Monitoring](#-monitoring)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🔧 Prerequisites

### Software Requirements

- **Python** 3.8 or higher
- **Apache Kafka** 2.8+ (with Zookeeper)
- **AWS Account** with appropriate permissions
- **AWS CLI** configured with credentials

### Python Dependencies

```bash
kafka-python>=2.0.2
boto3>=1.26.0
pandas>=1.5.0
python-dotenv>=0.19.0
```

### AWS Services Required

- AWS S3 (Storage)
- AWS Glue (Metadata Catalog)
- AWS Athena (Query Engine)
- IAM (Access Management)

---

## 📦 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/thetorangi/real-time-stock-data-pipeline-with-kafka.git
cd real-time-stock-data-pipeline-with-kafka
```

### 2. Create Virtual Environment

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Install Kafka

**Ubuntu/Debian:**
```bash
wget https://downloads.apache.org/kafka/3.5.0/kafka_2.13-3.5.0.tgz
tar -xzf kafka_2.13-3.5.0.tgz
cd kafka_2.13-3.5.0
```

**macOS (using Homebrew):**
```bash
brew install kafka
```

---

## ⚙️ Configuration

### 1. Environment Variables

Create a `.env` file in the project root:

```bash
# Kafka Configuration
KAFKA_BOOTSTRAP_SERVERS=localhost:9092
KAFKA_TOPIC=stock-market-data

# AWS Configuration
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
S3_BUCKET_NAME=stock-data-pipeline
S3_PREFIX=raw-data/

# Glue Configuration
GLUE_DATABASE=stock_market_db
GLUE_TABLE=stock_data

# Data Source
DATA_SOURCE_PATH=./data/stock_data.csv
```

### 2. Kafka Configuration

Edit `config/kafka_config.py`:

```python
KAFKA_CONFIG = {
    'bootstrap_servers': ['localhost:9092'],
    'client_id': 'stock-producer',
    'compression_type': 'gzip',
    'max_in_flight_requests_per_connection': 5,
    'enable_idempotence': True
}

CONSUMER_CONFIG = {
    'bootstrap_servers': ['localhost:9092'],
    'group_id': 'stock-consumer-group',
    'auto_offset_reset': 'earliest',
    'enable_auto_commit': True
}
```

---

## 🚀 Usage

### Step 1: Start Kafka Services

```bash
# Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka Server (in a new terminal)
bin/kafka-server-start.sh config/server.properties

# Create Kafka Topic
bin/kafka-topics.sh --create \
  --topic stock-market-data \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1
```

### Step 2: Run Kafka Producer

```bash
python producer/stock_producer.py
```

**Expected Output:**
```
[INFO] Kafka Producer started...
[INFO] Published: {"Index":"HSI","Date":"1986-12-31","Close":2568.30,...}
[INFO] Published: {"Index":"AAPL","Date":"2020-01-02","Close":300.35,...}
```

### Step 3: Run Kafka Consumer

```bash
python consumer/stock_consumer.py
```

**Expected Output:**
```
[INFO] Kafka Consumer started...
[INFO] Consumed message from partition 0
[INFO] Transformed and uploaded to S3: s3://bucket/index=HSI/year=1986/month=12/
```

### Step 4: AWS Glue Crawler

```bash
python scripts/run_glue_crawler.py
```

### Step 5: Query with Athena

```bash
python scripts/athena_query.py
```

---

## 📊 Dataset Schema

### Sample Record

```csv
Index,Date,Open,High,Low,Close,Adj Close,Volume,CloseUSD
HSI,1986-12-31,2568.30,2568.30,2568.30,2568.30,2568.30,0.0,333.88
AAPL,2020-01-02,296.24,300.60,295.30,300.35,297.43,135480400,300.35
```

### Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| **Index** | String | Stock ticker symbol (HSI, AAPL, TSLA, etc.) |
| **Date** | Date | Trading date (YYYY-MM-DD) |
| **Open** | Float | Opening price |
| **High** | Float | Highest price during trading |
| **Low** | Float | Lowest price during trading |
| **Close** | Float | Closing price |
| **Adj Close** | Float | Adjusted closing price |
| **Volume** | Float | Trading volume |
| **CloseUSD** | Float | Closing price in USD |

### S3 Partitioning Structure

```
s3://stock-data-pipeline/
└── raw-data/
    ├── index=HSI/
    │   ├── year=1986/
    │   │   └── month=12/
    │   │       └── data.parquet
    │   └── year=1987/
    │       ├── month=01/
    │       └── month=02/
    └── index=AAPL/
        └── year=2020/
            ├── month=01/
            └── month=02/
```

---

## 🔄 Pipeline Workflow

### 1. **Data Ingestion (Kafka Producer)**

```python
# producer/stock_producer.py
from kafka import KafkaProducer
import json

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

for record in read_stock_data():
    producer.send('stock-market-data', value=record)
```

### 2. **Stream Processing (Kafka Consumer)**

```python
# consumer/stock_consumer.py
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'stock-market-data',
    bootstrap_servers=['localhost:9092'],
    group_id='stock-consumer-group'
)

for message in consumer:
    data = json.loads(message.value)
    cleaned_data = transform_data(data)
    upload_to_s3(cleaned_data)
```

### 3. **Data Transformation Steps**

- ✅ Remove null/missing values
- ✅ Convert timestamps to UTC
- ✅ Validate data types
- ✅ Apply business logic
- ✅ Partition by index and date

### 4. **Storage & Cataloging**

- Data stored in **Parquet format** for efficiency
- **AWS Glue Crawler** infers schema automatically
- **Metadata** stored in Glue Data Catalog

---

## ☁️ AWS Setup

### 1. Create S3 Bucket

```bash
aws s3 mb s3://stock-data-pipeline --region us-east-1
```

### 2. Create Glue Database

```bash
aws glue create-database \
  --database-input '{"Name":"stock_market_db","Description":"Stock market data"}'
```

### 3. Create Glue Crawler

```bash
aws glue create-crawler \
  --name stock-data-crawler \
  --role AWSGlueServiceRole \
  --database-name stock_market_db \
  --targets '{"S3Targets":[{"Path":"s3://stock-data-pipeline/raw-data/"}]}'
```

### 4. Run Glue Crawler

```bash
aws glue start-crawler --name stock-data-crawler
```

### 5. IAM Permissions Required

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "glue:*",
        "athena:*"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 🔍 SQL Queries

### Query 1: Average Closing Price

```sql
SELECT 
    Index,
    AVG(Close) as avg_close_price
FROM stock_market_db.stock_data
WHERE year = '2020'
GROUP BY Index
ORDER BY avg_close_price DESC;
```

### Query 2: Top Trading Volume Days

```sql
SELECT 
    Index,
    Date,
    Volume
FROM stock_market_db.stock_data
WHERE Index = 'AAPL'
ORDER BY Volume DESC
LIMIT 10;
```

### Query 3: Yearly Trend Analysis

```sql
SELECT 
    Index,
    year,
    COUNT(*) as trading_days,
    AVG(Close) as avg_close,
    MAX(High) as max_high,
    MIN(Low) as min_low
FROM stock_market_db.stock_data
GROUP BY Index, year
ORDER BY Index, year;
```

### Query 4: Volatility Calculation

```sql
SELECT 
    Index,
    Date,
    (High - Low) / Open * 100 as daily_volatility_pct
FROM stock_market_db.stock_data
WHERE Index = 'TSLA' 
  AND year = '2021'
ORDER BY daily_volatility_pct DESC;
```

---

## 📈 Monitoring

### Kafka Monitoring

```bash
# Check topic lag
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group stock-consumer-group

# Monitor broker metrics
kafka-run-class.sh kafka.tools.JmxTool \
  --object-name kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec
```

### AWS CloudWatch Metrics

- S3 bucket size and object count
- Athena query execution times
- Glue crawler run status

### Python Logging

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('pipeline.log'),
        logging.StreamHandler()
    ]
)
```

---

## 🐛 Troubleshooting

### Issue: Kafka Connection Refused

**Solution:**
```bash
# Check if Kafka is running
ps aux | grep kafka

# Verify Zookeeper is running
telnet localhost 2181

# Check firewall rules
sudo ufw status
```

### Issue: S3 Access Denied

**Solution:**
```bash
# Verify AWS credentials
aws sts get-caller-identity

# Check bucket permissions
aws s3api get-bucket-policy --bucket stock-data-pipeline
```

### Issue: Glue Crawler Fails

**Solution:**
- Verify IAM role has S3 read permissions
- Check S3 path exists and contains data
- Ensure data format is consistent

### Issue: Athena Query Timeout

**Solution:**
- Increase query timeout in Athena settings
- Optimize partitioning strategy
- Use columnar format (Parquet) instead of CSV

---

## 📂 Project Structure

```
stock-data-pipeline/
├── config/
│   ├── kafka_config.py
│   └── aws_config.py
├── producer/
│   └── stock_producer.py
├── consumer/
│   └── stock_consumer.py
├── scripts/
│   ├── run_glue_crawler.py
│   └── athena_query.py
├── data/
│   └── stock_data.csv
├── tests/
│   ├── test_producer.py
│   └── test_consumer.py
├── docs/
│   ├── architecture.md
│   └── deployment.md
├── .env.example
├── .gitignore
├── requirements.txt
├── README.md
└── LICENSE
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. **Fork the repository**
2. **Create a feature branch** (`git checkout -b feature/amazing-feature`)
3. **Commit your changes** (`git commit -m 'Add amazing feature'`)
4. **Push to the branch** (`git push origin feature/amazing-feature`)
5. **Open a Pull Request**

### Code Style

- Follow PEP 8 for Python code
- Use meaningful variable names
- Add docstrings to functions
- Write unit tests for new features

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 📚 Additional Resources

- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [AWS Glue Developer Guide](https://docs.aws.amazon.com/glue/)
- [AWS Athena User Guide](https://docs.aws.amazon.com/athena/)
- [Kafka Python Client](https://kafka-python.readthedocs.io/)
- [Boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)

---

## 👥 Authors

- **coderKarma** -- [https://github.com/thetorangi](https://github.com/thetorangi)

---


<div align="center">

**⭐ Star this repository if you find it helpful!**

Made with ❤️ by the Data Engineering Team

</div>

# 📘 Report: Hands-on #11 – AWS EC2, Spark & Docker Web App

**Submitted by:** Prakathesh  
**Student Email:** pkumaras@uncc.edu

---

## ✅ Objective

This assignment focused on:

- Running a PySpark word count on AWS EC2 with S3 integration  
- Creating and deploying a Dockerized Node.js web application  
- Learning cloud-based big data processing and container deployment  

---

## 🔹 TASK 1: Spark Word Count on EC2 + S3

### ⚙️ EC2 Setup (Amazon Linux 2)

```bash
ssh -i "wordcount.pem" ec2-user@<EC2_PUBLIC_IP>
```

- Connects to the EC2 instance using SSH and your private key.

```bash
sudo yum update -y
sudo yum install java-11 -y
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
java --version
sudo mount -o remount,size=2G /tmp
sudo yum install python3-pip -y
pip3 install pyspark
```

### ⚙️ PySpark Word Count Script

**File:** `word_count.py`

```python
from pyspark.sql import SparkSession

# AWS Credentials
AWS_ACCESS_KEY_ID = 'YOUR_ACCESS_KEY'
AWS_SECRET_ACCESS_KEY = 'YOUR_SECRET_KEY'

# S3 paths
S3_INPUT = 's3a://wordcountprakathesh/input.txt'
S3_OUTPUT = 's3a://wordcountprakathesh/output/'

# Spark Session
spark = SparkSession.builder     .appName("WordCount")     .config("spark.jars.packages", "org.apache.hadoop:hadoop-aws:3.3.1,com.amazonaws:aws-java-sdk-bundle:1.11.901")     .getOrCreate()

# Hadoop S3 Configuration
hadoop_conf = spark.sparkContext._jsc.hadoopConfiguration()
hadoop_conf.set("fs.s3a.access.key", AWS_ACCESS_KEY_ID)
hadoop_conf.set("fs.s3a.secret.key", AWS_SECRET_ACCESS_KEY)

# Word Count Logic
text_file = spark.sparkContext.textFile(S3_INPUT)
counts = text_file.flatMap(lambda line: line.split())                   .map(lambda word: (word, 1))                   .reduceByKey(lambda a, b: a + b)
counts.saveAsTextFile(S3_OUTPUT)
spark.stop()
```

- Reads a text file from S3, performs word count using PySpark, and writes the output back to S3.

---

## 🔹 TASK 2: Dockerized Node.js Web App

### 📁 Create Node.js App

```bash
mkdir node-webserver && cd node-webserver
npm init -y
npm install express
```

**File:** `server.js`

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
    res.send('Hello, World! Running in Docker.');
});

app.listen(port, () => console.log(`Server running at http://localhost:${port}`));
```

### 🐳 Dockerfile

```Dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### 🔧 Docker on EC2

```bash
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user
logout  # then re-login for changes to take effect
```

### 🛠️ Build & Run Container

```bash
docker build -t prakathesh/webserver:latest .
docker run -d -p 80:3000 prakathesh/webserver:latest
```

### 🌐 Access App

```bash
curl http://localhost
curl http://checkip.amazonaws.com
```

- Visit in browser: `http://<EC2_PUBLIC_IP>`  
✔️ You should see: `"Hello, World! Running in Docker."`

---

## 📤 DockerHub Push

```bash
docker login
docker tag prakathesh/webserver:latest prakathesh/webserver:latest
docker push prakathesh/webserver:latest
```

---

## 🔧 Troubleshooting Done

- ❌ Output folder already exists → manually deleted S3 output folder  
- 🛑 Removed AWS keys before pushing to GitHub  
- ⚠️ Merge conflicts resolved using `--allow-unrelated-histories`

---

## 🔗 Links
 
- **Web App (EC2):** http://34.226.147.146/

---

## ✅ Submission Checklist

- [x] EC2 instance created  
- [x] PySpark word count completed  
- [x] Docker container built and deployed  
- [x] DockerHub image pushed  
- [x] GitHub repo cleaned and submitted  
- [x] Final report written  



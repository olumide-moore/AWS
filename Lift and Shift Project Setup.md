## Project Overview

This is a **lift and shift** deployment of the **vProfile** multi-tier web application to AWS Cloud. The apps consists of Tomcat app which uses backend services of MySQL, RabbitMQ, Memcached.

### AWS Services Used

* EC2 (Tomcat, MySQL, RabbitMQ, Memcached)
* ELB (Application Load Balancer)
* Auto Scaling Group
* Route 53 (Private DNS)
* ACM (SSL certificates)
* S3 (artifact storage)
* IAM, EBS, EFS (supporting services)

---

## Setup Workflow

1. **Log in to AWS**
2. **Create Key Pairs** (for EC2 SSH access)
3. **Create Security Groups** for:

   * Load Balancer
   * Tomcat (allow port 8080 from ELB only)
   * Backend services (MySQL, RabbitMQ, Memcached)
4. **Launch EC2 Instances** with `UserData` (bash scripts)
5. **Configure Route 53** private hosted zone (name to IP mapping for backend)
6. **Build App Artifact** locally from source
7. **Upload Artifact to S3**
8. **Download Artifact on Tomcat EC2 from S3**
9. **Setup ELB** with HTTPS using ACM certificate
10. **Map ELB DNS to GoDaddy Domain**
11. **Verify Setup**
12. **Create Auto Scaling Group** for Tomcat instances

---


Here’s the `README.md` section for the security group configuration:

---

## Security Groups Configuration (Inbound rules)

### 1. **Load Balancer SG** (`vprofile-ELB-sg`) Allows IPV4 and IPV6 HTTP and HTTPS public access


| Type         | Protocol | Port | Source    |
| ------------ | -------- | ---- | --------- |
| HTTP         | TCP      | 80   | 0.0.0.0/0 |
| HTTPS        | TCP      | 443  | 0.0.0.0/0 |
| HTTP (IPv6)  | TCP      | 80   | ::/0      |
| HTTPS (IPv6) | TCP      | 443  | ::/0      |

#### Optional (for debugging):
### 2. **App (Tomcat) SG** (`vprofile-app-sg`) Allows ELB sg and my Ip ssh (optionally My Ip on 8080 port for verifying the app ip can be loaded on internet)

| Type       | Protocol | Port | Source       |               |
| ---------- | -------- | ---- | ------------ |  ------------ |
| Custom TCP | TCP      | 8080 | `elb sg` |
| SSH        | TCP      | 22   | `my-ip/32` |    
| Custom TCP | TCP      | 8080 | `my-ip/32` | optional |

---

### 3. **Backend SG** (`vprofile-backend-sg`) for MySQL, RabbitMQ, Memcached

| Type         | Protocol | Port  | Source                     |
| ------------ | -------- | ----- | -------------------------- |
| All traffic  | All      | All   | vprofile-backend-sg (self) |
| SSH          | TCP      | 22    | `my-ip/32`               |
| Custom TCP   | TCP      | 5672  | vprofile-Tomcat-sg         |
| Custom TCP   | TCP      | 11211 | vprofile-Tomcat-sg         |
| MySQL/Aurora | TCP      | 3306  | vprofile-Tomcat-sg         |

---

## Launching EC2 Instances

Set up four separate EC2 instances for the vProfile app stack using **User Data scripts**:

1. **MySQL (MariaDB)**
2. **Memcached**
3. **RabbitMQ**
4. **Tomcat (App Server)**

#### **Source Code**

* Repository: `https://github.com/hkhcoder/vprofile-project`
* Branch: `aws-lift-and-shift`

---

### **Step-by-Step Setup**

#### 1. **Clone Source Code**

* Clone the repo into your local dev environment (e.g., VS Code).
* Switch to `aws-lift-and-shift` branch.

#### 2. **EC2 Setup (Repeated Steps for Each Instance)**

* **AMI**: Amazon Linux 2023 for all except Tomcat (use Ubuntu 24 LTS for Tomcat)
* **Instance Type**: T2 Micro / T3 Micro (free tier)
* **Key Pair**: `vprofile-prod-key`
* **Tags**: Add meaningful tags, including `Name` and `Project: vprofile`, Include `Volume` in Resource tag
* **Security Groups**:

  * **Backend SG**: For MySQL, Memcached, RabbitMQ
  * **App SG**: For Tomcat instance

#### 3. **User Data Scripts**

Paste the relevant script from `userdata/` folder during instance launch (in Advanced details):

* `mysql.sh`
* `memcache.sh`
* `rabbitmq.sh`
* `tomcat_ubuntu.sh`

---

### **Post-Launch Validation**

After instance creation:

* **SSH into each instance** using correct username:
  * `ec2-user` for Amazon Linux
  * `ubuntu` for Ubuntu

* **Verify Services**:

  * `systemctl status mariadb`
  * `systemctl status memcached`
  * `systemctl status rabbitmq-server`
  * `systemctl status tomcat10`

* **Check MySQL DB Setup**:
  * Login as: `mysql -u admin -p`
  * DB Name: `accounts`
  * Verify tables: `show tables;`

---


Certainly! Here's a **concise, structured README-style note** based on the transcript you provided, with all emojis and troubleshooting removed, and focused on clarity and key steps:

---

## Internal DNS Setup Using Route 53 (AWS)

The `app01` instance (Tomcat server) needs to connect to three backend services:

* MySQL Database (`db01`)
* Memcache (`mc01`)
* RabbitMQ (`rmq01`)

To ensure scalable and maintainable configuration, use **hostnames** instead of hardcoding **private IP addresses** in the application config.

---

### 1. Application Configuration

Edit:
`src/main/resources/application.properties`

Use **hostnames**, not IP addresses:

```
db01.vprofile.in
mc01.vprofile.in
rmq01.vprofile.in
```

---

### 2. DNS Resolution with Route 53

Use **Route 53 Private Hosted Zone** to resolve hostnames to private IPs.

### Steps:

1. **Go to Route 53 > Create Hosted Zone**

   * Domain Name: `vprofile.in`
   * Type: **Private Hosted Zone**
   * Region: e.g., `us-east-1 (N. Virginia)`
   * VPC: Select default VPC

2. **Create A Records** (used for` name → IP` mapping)

| Record Name | Type | Value (Private IP)                    |
| ----------- | ---- | ------------------------------------- |
| `db01`      | A    | \[Private IP of `db01`]               |
| `mc01`      | A    | \[Private IP of `mc01`]               |
| `rmq01`     | A    | \[Private IP of `rmq01`]              |
| `app01`     | A    | \[Private IP of `app01`] *(optional)* |

---

### 3. Validate DNS Resolution

From the `app01` instance (Ubuntu AMI):

```bash
ping -c 4 db01.vprofile.in
ping -c 4 mc01.vprofile.in
ping -c 4 rmq01.vprofile.in
```
Ensure IP matches the correct private IP of the target instance

---

**Build App Artifact** locally from source
7. **Upload Artifact to S3**
8. **Download Artifact on Tomcat EC2 from S3**
## Build and Upload App to AWS EC2 (Tomcat) via S3
A Java `.war` artifact is built locally using Maven, uploaded to an S3 bucket, and downloaded to the Tomcat instance (`app01`) using AWS CLI.

---


### Prerequisites 
(On Local Machine)
* **Maven 3.9.9** (check both maven and java with `mvn --version`)
* **JDK 17** (check with `java --version`)
* **AWS CLI** (check with aws)

* `application.properties` correctly configured with:
  * `db01.vprofile.in`
  * `mc01.vprofile.in`
  * `rmq01.vprofile.in`

* IAM user with S3 Full Access (On AWS)

#### EC2 Tomcat Server (`app01`)
  * **AWS CLI installed (via `snap`)**
  * **IAM Role with S3 Full Access** attached

### 1. Create AWS Resources

#### S3 Bucket

* Go to **S3** in AWS Console
* Create a bucket (e.g., `vprofile-<unique>-artifacts`)

#### IAM User (For Local Machine)

* Service: **IAM → Users → Create User**
* Name: `vprofile-s3-admin`
* Attach Policy: `AmazonS3FullAccess`
* Select created user and under `Security Credentials`-> `Create Access Key` for `Command Line Interface (CLI)` and save it

#### IAM Role (For EC2)

* Service: **IAM → Roles → Create Role**
* Trusted Entity: `EC2`
* Permissions: `AmazonS3FullAccess`
* Name: `s3-admin`
* Attach role to the Tomcat App EC2 via:
  `EC2 → Select instance → Actions → Security → Modify IAM Role`
---

### 2. Prepare Local Machine

```bash
# Configure AWS CLI with downloaded keys
aws configure
# Input Access Key, Secret Key, Region, Output: json

#FYI: Inputs can be changed in :
ls ~/.aws/credentials
ls ~/.aws/config
```

---

### 3. Build Artifact with Maven

```bash
# Navigate to source code root
cd <source_code_dir>

# Build .war file
mvn install

# Check if target/vprofile-v2.war is generated
```

---

### 4. Upload Artifact to S3

```bash
# Upload to S3 bucket
aws s3 cp target/vprofile-v2.war s3://<your-bucket-name>/
```

---

### 5. EC2 Instance: Fetch & Deploy Artifact

```bash
# SSH into Tomcat app EC2
ssh -i <your-key.pem> ubuntu@<public-ip>

# Switch to root
sudo -i

# Install AWS CLI
snap install aws-cli --classic

# Copy .war from S3
aws s3 cp s3://<your-bucket-name>/vprofile-v2.war /tmp/

# Stop Tomcat
systemctl stop tomcat10
systemctl daemon-reload  # If necessary

# Remove default ROOT app
rm -rf /var/lib/tomcat10/webapps/ROOT

# Copy and rename .war to ROOT.war
cp /tmp/vprofile-v2.war /var/lib/tomcat10/webapps/ROOT.war

# Start Tomcat
systemctl start tomcat10

# App should extract and become accessible
```

---

### Validation

* Navigate to: `http://<EC2-public-ip>:8080`(if your security group includes access to 8080 with My IP)
* App should load from Tomcat using deployed artifact

---

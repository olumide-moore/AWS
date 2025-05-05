# AWS Free Tier Account Setup Guide  

This guide walks you through setting up an AWS Free Tier account securely. It includes configuring IAM with MFA, enabling billing alerts, and requesting an SSL certificate from AWS ACM.

---

## 1. Create an AWS Free Tier Account  

1. Visit [AWS Free Tier](https://aws.amazon.com/free/).  
2. Click **"Create a Free Account"** and follow the registration steps.  
3. Provide payment details (AWS requires a credit/debit card, but free-tier usage won't be charged).  
4. Complete phone verification and sign in to the AWS Management Console.  

---

## 2. Set Up IAM User and Enable MFA  

### Enable MFA for the Root User  
1. Log in to AWS as the **Root User**.  
2. Navigate to **IAM** → Click **Security Recommendations**.  
3. Click **"Assign MFA"**, select **Authenticator App**, and follow the instructions.  
4. Scan the QR code with **Google Authenticator** or **Authy**, enter the generated codes, and complete the setup.  

### Create an IAM User with Admin Access  
1. Go to **IAM** → **Users** → **Add users**.  
2. Enter a **User Name** (e.g., `admin`).  
3. Enable **AWS Management Console Access** and set a **password**.  
4. Attach the **AdministratorAccess** policy.  
5. Download the credentials CSV file (contains login URL, username, and password).  

### Enable MFA for IAM User  
1. Sign in as the IAM user.  
2. Navigate to **IAM** → Select the user → **Security Credentials**.  
3. Click **"Assign MFA"** and follow the steps.  


### Create an IAM Login Alias  
1. Go to **IAM Dashboard** → **Account Settings**.  
2. Click **Create Account Alias** and enter a unique alias (e.g., `mycompany`).  
3. Save the changes.  
4. The new login URL will be:  

---

## 3. Set Up Billing Alerts  

### Enable Billing Alerts  
1. Open **Billing Dashboard** → **Billing Preferences**.  
2. Enable:  
   - **"Receive AWS Free Tier Alerts"**  
   - **"Receive CloudWatch Billing Alarms"**  
3. Enter an email for notifications and save preferences.  

### Create a CloudWatch Billing Alarm  
1. Open **CloudWatch** → **Alarms** → **Create Alarm**.  
2. Select **Billing Metrics** → **Total Estimated Charges** in **USD**.  
3. Set a **Threshold** (e.g., `$5`).  
4. Create an **SNS Topic** to send email notifications.  
5. Confirm the SNS subscription via email and activate the alarm.  

---

## 4. Request an SSL Certificate from AWS ACM  

### Why Use HTTPS?  
Whenever you see **HTTPS** in a website’s URL, a padlock icon appears in the browser. This means the connection is secure and validated by an SSL certificate. AWS provides free certificates through **AWS Certificate Manager (ACM)**.  

### Request an SSL Certificate  
1. Open **AWS Certificate Manager (ACM)**.  
2. Click **"Request a Certificate"** → **Request a Public Certificate**.  
3. Enter your domain (e.g., `yourdomain.com`).  
4. Use **DNS Validation** (recommended) or **Email Validation**.  
5. Click **Request** and review the certificate status.  

### Validate the Certificate Using DNS  
1. In ACM, find the **CNAME Record** required for validation.  
2. Log in to your **domain registrar** (e.g., GoDaddy).  
3. Go to **Manage Domain** → **DNS Settings**.  
4. Add a new **CNAME Record**:  
   - **CNAME Name**: Copy from ACM (remove the domain part).  
   - **CNAME Value**: Copy the full value from ACM (remove the . at the end).  
5. Save the record.  

AWS will now check if your domain points to the correct CNAME. This process can take up to **48 hours**. Once validated, the certificate will show as **Issued**.  

---


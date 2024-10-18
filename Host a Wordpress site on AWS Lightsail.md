
# AWS projects

## Host a Wordpress site on AWS Lightsail

#### Step 1: Create a domain name via Route 53
#### Step 2: Create a Lightsail instance with Wordpress
#### Step 3: To access the Wordpress admin panel
- Step 3.1: Open the Lightsail terminal and run the following command to fetch admin password
```bash
cat bitnami_application_password  # This will output the password from the file
```
- Step 3.2: Navigate public IP address in the path `/wp-admin`, then login to Wordpress admin with  username `user` and password from previous step
- Step 3.3: Change the password to a memorable one in Wordpress 'Users' settings

#### Step 4: Connect your Route 53 domain name to the Lightsail instance
- Step 4.1: Navigate to Lightsail networking section and create a static IP address (this will replace the dynamic IP address)
- Step 4.2: Navigate to Route 53 Hosted Zones and create a new record with the static IP address as the value
- Step 4.3: Create a secord record - CNAME with the domain name as the value (this might take a few minutes/hours to propagate)

#### Step 5: Install an SSL certificate (this will make your site secure with HTTPS)
- Step 5.1: Navigate to the Lightsail instance and click on the 'Networking' tab
- Step 5.2: Click on 'Create SSL certificate' and follow the instructions to create a certificate
- Step 5.3: Once the certificate is created, click on 'Attach to a Lightsail instance' and select the instance you want to attach it to
- Step 5.4: Navigate to the 'Networking' tab and click on 'Create DNS zone' to create a new DNS zone
- Step 5.5: Add a new record with the domain name and the SSL certificate as the value
- Step 5.6: Add a second record for www.yourdomain.com and point it to the same SSL certificate (this might take a few minutes/hours to propagate)

##### If you want email hosting with your domain, you can use AWS WorkMail (https://youtu.be/g3nDQ0-jxSU?si=xHguKIVlaXA6qhlJ) or a third-party service like Zoho Mail. This will invoive you creating an MX record and a TXT record in Route 53

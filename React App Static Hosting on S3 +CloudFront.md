
# AWS projects

## React App on AWS S3 with Static Hosting + Cloudfront
    
### Create your React App
1. Create a new React app using `create-react-app`:
    ```bash
    npx create-react-app my-app
    cd my-app
    npm start
    ```
2. Build the app (after you have completed the react code):
    This will bundle the files into static files in the `build` folder for production.
    ```bash
    npm run build
    ```

### Create an S3 bucket
1. Create one bucket with 'www' as the starting name (e.g. www.my-app.com).
2. Creat a second bucket with the same name as the first bucket but without the 'www' (e.g. my-app.com).

### Upload the build folder to the 'www' bucket
1. Go to the 'www' bucket and upload all the files within the `build` folder.
2. Upload the static folder within the `build` folder.

### Enable public access to the 'www' bucket
1. Go to the 'www' bucket and click on the 'Permissions' tab.
2. Edit the 'Block public access' settings and uncheck the 'Block all public access' box.
3. Edit the bucket policy and add the following policy:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::[www-bucket-name]/*"
            }
        ]
    }
    ```

### Enable static website hosting on the 'www' and the second 'app' bucket
1. Go to the buckets and click on the 'Properties' tab.
2. Click on 'Static website hosting' and enable it.
3. For the 'www' bucket, set hosting type to 'Host a static website' and set the index document to 'index.html'.
4. For the 'app' bucket, set hosting type to 'Redirect requests' and set the target bucket or domain to the 'www' bucket. (initially set protocol to 'http' and then change it to 'https' after creating the CloudFront distribution).

### Create record in Route 53
1. Go to Route 53 and create a new record set. (assuming you have already registered a domain)
2. Set the record name to 'www' and the record type to 'A - IPv4 address'.
3. Set the alias target to 'Alias to S3 website endpoint' and select the S3 region and'www'  bucket.
4. Uncheck the 'Evaluate target health' box.
5. Click on 'Define simple record' and create the second record set with the same name as the first record set but without the 'www' (e.g. my-app.com). Then create both record sets.

### --By this point, you should be able to access your React app using the domain name you registered (however, it will not be secure).--
# CloudFront

## CloudFront overview

### CloudFront origin 

* S3 bucket 
* **Any** HTTP backend, such as your on premises infrastructure

### Origin Access Identity

Restricting access to Amazon S3 content by using an **origin access identity (OAI)**

To restrict access to content that you serve from Amazon S3 buckets, follow these steps:

1. Create a special CloudFront user called an origin access identity (OAI) and associate it with your distribution.
2. Configure your S3 bucket permissions so that CloudFront can use the OAI to access the files in your bucket and serve them to your users. Make sure that users can’t use a direct URL to the S3 bucket to access a file there.

## Hands On - Access S3 bucket content from CloudFront

* **Create an S3 bucket**

  Bucket name: content-for-cloudfront18

  Region: us-east-1

* **Create a CloudFront distribution**

  Distribution domain name: https://d12l3kjhqdv0x4.cloudfront.net

  Creation of cloudFront distribution is super fast

* **Create an Origin Access Identity**

  During create CloudFront process, we can select create OAI, then AWS will create one for us. 

  OAI ID: E10VADW9QKOMYP. you can check in Origin Access Identity menu of CloudFront console. 

* **Limit the S3 bucket to be accessed only using this identity**

  - **Block all public access** is checked. 

    We need to make sure CloudFront can READ from this S3 bucket but **there should be absolutely NO public access to this bucket**. This is important because a public accessible S3 bucket allows end user to bypass CloudFront to access restricted file directly.

  - **Bucket policy**: 

    Only **OAI** E10VADW9QKOMYP allows to get objects from bucket content-for-cloudfront18

    Note the principle is CloudFront user **CloudFront Origin Access Identity [ID]**

      ```
      {
          "Version": "2008-10-17",
          "Id": "PolicyForCloudFrontPrivateContent",
          "Statement": [
              {
                  "Sid": "1",
                  "Effect": "Allow",
                  "Principal": {
                      "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity E10VADW9QKOMYP"
                  },
                  "Action": "s3:GetObject",
                  "Resource": "arn:aws:s3:::content-for-cloudfront18/*"
              }
          ]
      }
      ```

* **Testing**:

  I uploaded coffee.jpg file to the S3 bucket.

  Then give CloudFront url: https://d12l3kjhqdv0x4.cloudfront.net/coffee.jpg. I can see the image. 

## CloudFront Reports, Logs and Troubleshooting

### How to turn on Access logs

CloudFront console > distribution > Select a distribution> Setting> Edit > Standard logging: ON, then give a S3 bucket for storing the logs. 

### CloudFront reports

* Cache Statistics Report: percentage of cache hit
* Popular Objects Report: total request for a objects, percentage of cache hit
* Top Referrers Report: From which page get the most requests
* Usage Reports: How much data transferred from CloudFront to Client, how much data transferred from CloudFront to Client
* Viewers Report: What type of viewer (Desktop mobil), view location (country)

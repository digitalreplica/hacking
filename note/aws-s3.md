---
is_a: "[[note]]"
topics: "[[hacking aws]]"
---
# Notes
AWS S3

Listing all buckets
```
aws s3api list-buckets --query "Buckets[].Name"
```

Copy file to a bucket
```
aws s3 cp myfile.txt s3://mybucket/myfile.txt
```

Sync a folder with a bucket
- https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html
```
aws s3 sync . s3://mybucket
```

Check what files will be sync'd
```
aws s3 sync --dryrun . s3://mybucket
```
## REST endpoints
Virtual hosted-style dual-stack endpoint:
    {bucketname}.s3.{aws-region}.amazonaws.com
Path-style dual-stack endpoint:
    s3.{aws-region}.amazonaws.com/{bucketname}

## Dualstack IPv6 REST endpoints
Virtual hosted-style dual-stack endpoint:
    {bucketname}.s3.dualstack.{aws-region}.amazonaws.com
Path-style dual-stack endpoint:
    s3.dualstack.{aws-region}.amazonaws.com/{bucketname}

## Static Website endpoints
s3-website dash (-) Region ‐ http://{bucket-name}.s3-website-{aws-region}.amazonaws.com
s3-website dot (.) Region ‐ http://{bucket-name}.s3-website.{aws-region}.amazonaws.com

* Look up all names via DNS, see if they're CNAMEs to buckets (or ELBs)

## Access point endpoints
* additional research needed. see https://docs.aws.amazon.com/AmazonS3/latest/dev/using-access-points.html

# Questions
* How to search for buckets accessible only via a private S3 endpoint?

# Documentation
[Developer Guide](https://docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html)
[Rest API endpoints](https://docs.aws.amazon.com/general/latest/gr/s3.html)
[Amazon S3 ARN Format](https://docs.aws.amazon.com/AmazonS3/latest/dev/s3-arn-format.html)

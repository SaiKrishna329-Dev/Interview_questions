### 1. how to configure app in account A to access S3 in account B?
**Ans:**
- create IAM role in account B to access S3 and in that role's trust policy add the account A ID       
- copy the role ARN (arn:aws:iam::<AccountB_ID>:role/CrossAccountS3AccessRole) 
         
- attach the IAM policy to the app's IAM role/user as AssumeRole in account B --> run
  ``` 
  aws sts assume-role \
    --role-arn arn:aws:iam::<AccountB_ID>:role/CrossAccountS3AccessRole \
    --role-session-name AppAccessS3
    ```
     gives temporary creds as accesskey, secretkey and secure token,we can use these creds to access but better sol to use AWS SDK for this task like python code.

### 2. Can we change the private ip of instance either its running or stopped?why ?   
**Ans:** 
- Primary private IP → fixed to the Elastic Network Interface(ENI); can’t change directly (only by replacing ENI).
**Reason:**
  - The private IP is bound to the ENI (which is the true network identity of an EC2 instance).
  - AWS uses that mapping for routing inside the VPC.
  - Allowing dynamic change of primary IP would break routing consistency and DNS mappings.
- Secondary private IPs → can add/remove/change anytime.
- Stop/start doesn’t change primary private IP, only public IP. 
### 3. what are backend services for lambda in aws?
**Ans:**
- AWS Lambda doesn’t have its own backend — it integrates with databases, storage, messaging, and API services in AWS.
- Typical backends are DynamoDB, S3, RDS, API Gateway, SQS/SNS, EventBridge depending on the use case.
### 4. Just after increasing the disk in aws console for a node will it reflect immediatly?
**Ans:**
- AWS Console resize → takes effect immediately at block level.
- Inside the OS → you must manually resize the partition/filesystem before the space becomes usable
  ``` 
  lsblk
  for partitioned: sudo growpart /dev/xvda 1
  for ext4 filesystem: sudo resize2fs /dev/xvda1
  for xfs filesystem: sudo xfs_growfs -d /
  ```
- No reboot needed for volume expansion.  
### 5. can we make public instance to private instance?
**Ans:**  Yes

**Option 1:** Remove Public IP
- Go to EC2 Console → Networking → Manage IP Addresses.
- Disassociate the Elastic IP (if attached).
- If it had an auto-assigned public IP → stop the instance, disable “Auto-assign public IPv4” in the subnet settings, then start it again.
- Now instance will only have a private IP → private instance.

**Option 2:** Move to a Private Subnet
- Create/identify a private subnet (only routes to NAT Gateway or internal VPC resources).
- Stop the instance.
- Change its Subnet via “Change Subnet” option or by attaching its ENI to the new subnet.
- Start instance.
### 6. S3 is regional or global? - Regional with global access
### 7. different types of policies in autoscaling and parameters for autoscaling? - target tracking, step scaling, scheduled, predective
 (https://chatgpt.com/share/6888274e-a934-8012-9e76-65977e6f2ba4)
### 8. s3 bucket, replication rule and lifecycle rule?
### 9. permission boundary in IAM?
### 10. what is default route in your main route table?
### 11. vpc peering how to configure and transit gateway?
### 12.  how to find and fix unhealthy target group in a ALP?   
### 13. Alternatives to NAT for Private Subnet Internet Access?
**Ans:** 
- VPC endpoints for services within the AWS services. (like S3, DynamoDB, ECR, SSM, Secrets Manager, etc.),    you can use: Gateway Endpoints (for S3 & DynamoDB),Interface Endpoints (PrivateLink) for other services.
- forward proxy - keeping proxy in public subnet and tranfer the route to proxy from private and get internet access through proxy.
- If EC2 only needs package updates or software installs, you can use SSM endpoints.(SSM Agent can download packages through SSM endpoints).
### 14. how to setup geolocation based routing using AWS services?
**Ans:** 
- Simple DNS-based routing (country-level only) → Route 53 Geolocation Routing.
 ```
 In Route 53 Hosted Zone, create multiple records for the same domain.
 Choose Routing Policy → Geolocation.
 Define location:
 Continent (e.g., Asia, Europe).
 Country (e.g., India, US).
 State (only for US).
 Point records to different AWS resources (ALBs, CloudFront, EC2, etc.).
 Add a Default record as fallback.
```
- Content customization, redirects, or WAF rules → CloudFront + Lambda@Edge. - CloudFront automatically gives you the viewer’s country code in the request headers (CloudFront-Viewer-Country).we can use this headers in Lambda@Edge to rewrite URLs or redirect users to country-specific apps.like Users from IN → redirect to /in/index.html
- Performance-optimized global routing → AWS Global Accelerator - It routes users to the closest healthy AWS endpoint based on geography and health checks.


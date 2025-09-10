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
### 3. what are backend services for lamnda in aws?
### 4. Just after increasing the disk in aws console for a node will it reflect immediatly?
### 5. can we make public instance to private instance?
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
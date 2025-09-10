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
### 6. S3 is regional or global? 
**Ans:**
- Regional with global access

| Aspect                    | Type             | Explanation                                         |
| ------------------------- | ---------------- | --------------------------------------------------- |
| Data storage              | **Regional**     | You select the region for your bucket.              |
| Access                    | **Global**       | Accessible from anywhere (if allowed).              |
| Bucket name uniqueness    | **Global**       | No two buckets in the world can have the same name. |
| Features like replication | **Cross-region** | Available if configured.                            |

### 7. different types of policies in autoscaling and parameters for autoscaling?
**Ans:**
 - target tracking, step scaling, scheduled, predective

 | Policy Type        | Triggers on         | Use Case                        | Complexity |
| ------------------ | ------------------- | ------------------------------- | ---------- |
| Target Tracking    | Metric target       | Simple, balanced workloads      | Easy       |
| Step Scaling       | Threshold + steps   | Custom thresholds and steps     | Medium     |
| Scheduled Scaling  | Fixed times         | Predictable, time-based traffic | Easy       |
| Predictive Scaling | Historical patterns | ML-based, anticipates traffic   | Advanced   |

 (https://chatgpt.com/share/6888274e-a934-8012-9e76-65977e6f2ba4)
### 8. s3 bucket, replication rule and lifecycle rule?
**Ans:**

| Feature         | Replication Rule                                   | Lifecycle Rule                                     |
| --------------- | -------------------------------------------------- | -------------------------------------------------- |
| Purpose         | Copy objects to another bucket                     | Manage object storage class & expiration           |
| Requirement     | Versioning must be enabled                         | Versioning optional (but useful for version rules) |
| Scope           | Works across buckets/regions                       | Works within same bucket                           |
| Common Use Case | Compliance, disaster recovery, multi-region copies | Cost optimization, data archival, auto-deletion    |

### 9. permission boundary in IAM?
**Ans:** 
- A permissions boundary is an IAM policy that sets the maximum allowed permissions for a user or role.
- Think of it as a fence 🪧 → the user/role can’t cross it, even if another admin tries to give them more permissions.It doesn’t grant permissions by itself — it only limits.
- uses: Ensure that even if someone attaches AdministratorAccess policy, the boundary stops it. When different teams/apps share an AWS account, boundaries prevent them from stepping on each other’s resources.
### 10. what is default route in your main route table?
**Ans:** 
- When you create a new VPC, the main route table by default has only one route: 
 Destination   Target
------------  --------
10.0.0.0/16   local

- Destination = VPC CIDR range → all traffic inside the VPC.
- Target = local → means traffic stays within the VPC (instances, subnets, etc.).
- This is called the default local route. It cannot be deleted.
- Ensures all subnets in the VPC can talk to each other by default
### 11. vpc peering how to configure and transit gateway?
**Ans:**
**Steps:**
- Create peering connection - VPC - Peering connections - Requester, Accepter
- Accept it - If both VPCs are in the same account → Accept from Console.
- Update route tables - Add routes in each VPC’s route table so that traffic can flow like VPC-A CIDR in VPC-B and vice-versa
- Update security groups - Add inbound/outbound rules to allow traffic from the peer VPC’s CIDR.

| Feature            | **VPC Peering**                | **Transit Gateway**                |
| ------------------ | ------------------------------ | ---------------------------------- |
| Best for           | Few VPCs, simple mesh          | Many VPCs, hub-and-spoke           |
| Transitive routing | ❌ No                           | ✅ Yes                              |
| Cost               | Cheaper (per data transfer)    | More expensive (TGW hourly + data) |
| Complexity         | Grows in full mesh (N^2 links) | Centralized & scalable             |

### 12.  how to find and fix unhealthy target group in a ALP?   
**Ans:**
- Go to EC2 → Target Groups → [Your TG] → Targets tab.
- You’ll see each target’s status: healthy, unhealthy, initial, unused, draining.

**Fix:**
- Health check port & path correct?
- Target SG allows ALB inbound traffic?
- NACL allows traffic both ways?
- App running & listening?
- Response code = expected success code?
- No startup delays / ASG misconfig?
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

### 15. what is cloudwatch logs and events?
**Ans:**
- CloudWatch Logs = "What happened inside the service/app (logs)?"
- CloudWatch Events = "Something happened in AWS, now react to it."

| Feature   | **CloudWatch Logs**                  | **CloudWatch Events / EventBridge**    |
| --------- | ------------------------------------ | -------------------------------------- |
| Purpose   | Store and monitor log data           | Respond to AWS service or app events   |
| Data Type | Log streams (textual logs, metrics)  | Event messages (JSON)                  |
| Trigger   | Manual/agent/application writes logs | AWS services emit events automatically |
| Example   | Error logs, access logs              | “EC2 stopped” → Trigger Lambda         |

### 16. Pre-Signed urls in S3?
**Ans:**
- A temporary URL generated using AWS credentials that grants time-limited access to an S3 object.
- Even if the bucket/object is private, you can share this URL so that others can download (GET) or upload (PUT/POST) files without needing AWS credentials.
```
# AWS CLI:
aws s3 presign s3://mybucket/myfile.txt --expires-in 60

# python script:
import boto3
import datetime

s3 = boto3.client('s3')

url = s3.generate_presigned_url(
    ClientMethod='get_object',
    Params={'Bucket': 'mybucket', 'Key': 'myfile.txt'},
    ExpiresIn=300  # 5 minutes
)

print("Download link:", url)
```
- secure, temporary, credential-free access to private S3 objects.

### 17. how to configure a private ec2 instance to fetch the objects from s3 and display in frontend?
**Ans:**
**Steps:**
- S3 Gateway Endpoint - create VPC endpoint for S3 - route traffic internally without NAT or internet
- If a VPC Endpoint is not used, you need a NAT Gateway in a public subnet - Private EC2 routes traffic 0.0.0.0/0 → NAT → IGW → S3.
- Create an IAM Role with permissions for S3 (least privilege) - Attach this IAM Role to your EC2 instance- now EC@ able to access the objects from S3.
- Place your private EC2 behind an Application Load Balancer (ALB) - ALB in a public subnet forwards traffic to EC2 in private subnet - User → ALB (public) → EC2 (private) → S3 (via VPC endpoint).

## 18. what is endpoint? and it types?
**Ans:**
| Feature            | Gateway Endpoint           | Interface Endpoint                             | GWLB Endpoint                      |
| ------------------ | -------------------------- | ---------------------------------------------- | ---------------------------------- |
| Supported Services | S3, DynamoDB               | Most AWS services + SaaS                       | Security appliances                |
| Cost               | Free                       | Charged per ENI + data                         | Charged                            |
| Traffic Routing    | Route Table                | Private IP (ENI)                               | GWLB in path                       |
| Use Case           | Private S3/DynamoDB access | Private API/SNS/SQS/KMS/Secrets Manager access | Insert firewall/inspection devices |

**Uses**
- Security: No public internet exposure.
- Compliance: Keep data in AWS private network.
- Cost optimization: Avoid NAT Gateway for S3/DynamoDB.
- Performance: Lower latency (regional access).
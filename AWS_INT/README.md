## 1. how to configure aapp in account A to access S3 in account B?
**Ans:**
 - create IAM role in account B to access S3 and in that role's trust policy add the account A ID       
 - copy the role ARN (arn:aws:iam::<AccountB_ID>:role/CrossAccountS3AccessRole) 
         
- attach the IAM policy to the app's IAM role/user as AssumeRole in account B --> run "aws sts assume-role \
    --role-arn arn:aws:iam::<AccountB_ID>:role/CrossAccountS3AccessRole \
    --role-session-name AppAccessS3" gives temporary creds as accesskey, secretkey and secure token,we can use these creds to access but better sol to use AWS SDK for this task like python code.
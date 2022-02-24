# tromzo-aws-integration

CloudFormation template to set up the AWS integration.

## Steps to configure the AWS integration

1. Ask the Tromzo team to generate an External ID.

2. Download the CloudFormation JSON from the following link
   https://github.com/tromzo/aws-integration/blob/main/tromzo_integration_iam_role.json.

3. Put the External ID into the following script and execute it:

   ```sh
   TROMZO_ACCOUNT_ID=<TROMZO_ACCOUNT_ID>
   TROMZO_EXTERNAL_ID=<EXTERNAL_ID_FROM_TROMZO>
   aws cloudformation create-stack \
       --stack-name tromzo-iam-integration \
       --capabilities="CAPABILITY_NAMED_IAM" \
       --template-body file://tromzo_integration_iam_role.json \
       --parameters \
       ParameterKey=TromzoAccountId,ParameterValue=$TROMZO_ACCOUNT_ID \
       ParameterKey=TromzoExternalId,ParameterValue=$TROMZO_EXTERNAL_ID
   ```
   If this is done in the CloudFormation web console, then remove the Outputs object entirely from the previously downloaded https://github.com/tromzo/aws-integration/blob/main/tromzo_integration_iam_role.json file.

4. Verify that the stack has completed successfully and then run the following command:

   ```sh
   aws iam get-role --role-name TromzoSecurityAuditRole
   ```

To complete the integration, provide the following pieces of information to us:

1. The ARN of the role TromzoSecurityAuditRole. It should look like this:
   `arn:aws:iam::<YOUR_ACCOUNT_ID>:role/TromzoSecurityAuditRole`.
2. The default region of your AWS account.

Tromzo CloudFormation template performs the following tasks:

1. Creates a role in your AWS account called TromzoSecurityAuditRole with the SecurityAudit policy attached to it. The
   SecurityAudit policy provides read-only access to resources in the AWS account.
2. Creates a Trust Relationship that allows a user named tromzo-security-audit-user in the Tromzo AWS account to assume
   the TromzoSecurityAuditRole in your account.

# tromzo-aws-integration

CloudFormation template to set up the AWS integration.

## Steps to configure the AWS integration in one account

1. Ask the Tromzo team to generate an External ID.

2. Download the CloudFormation JSON from the following link
   https://github.com/tromzo/aws-integration/blob/main/tromzo_integration_iam_role.json

3. Put the Tromzo Account ID and External ID into the following script and
   execute it:

   ```sh
   TROMZO_ACCOUNT_ID=<TROMZO_ACCOUNT_ID>
   TROMZO_EXTERNAL_ID=<EXTERNAL_ID_FROM_TROMZO>
   aws cloudformation create-stack \
       --stack-name tromzo-iam-integration \
       --capabilities CAPABILITY_NAMED_IAM \
       --template-body file://tromzo_integration_iam_role.json \
       --parameters \
       ParameterKey=TromzoAccountId,ParameterValue=$TROMZO_ACCOUNT_ID \
       ParameterKey=TromzoExternalId,ParameterValue=$TROMZO_EXTERNAL_ID
   ```

4. Verify that the stack has completed successfully by running the following
   command:

   ```sh
   aws iam get-role --role-name TromzoSecurityAuditRole
   ```

To complete the integration, provide the following pieces of information to us:

1. The ARN of the role TromzoSecurityAuditRole. It should look like this:
   `arn:aws:iam::<YOUR_ACCOUNT_ID>:role/TromzoSecurityAuditRole`.
2. The default region of your AWS account.

Tromzo CloudFormation template performs the following tasks:

1. Creates a role in your AWS account called TromzoSecurityAuditRole with the
   SecurityAudit policy attached to it. The SecurityAudit policy provides
   read-only access to resources in the AWS account.
2. Creates a Trust Relationship that allows a user named
   tromzo-security-audit-user in the Tromzo AWS account to assume the
   TromzoSecurityAuditRole in your account.

## Steps to configure the AWS integration in multiple accounts

We could use [AWS CloudFormation StackSets] to deploy the template to multiple
accounts.

[AWS CloudFormation StackSets]: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html

1. Ask the Tromzo team to generate an External ID.

2. Download the CloudFormation JSON from the following link
   https://github.com/TromsoSecurity/tromzo-aws-integration/raw/main/tromzo_integration_iam_role.json.

3. Put the Tromzo Account ID and External ID into the following script and
   execute it:

   ```sh
   TROMZO_ACCOUNT_ID=<TROMZO_ACCOUNT_ID>
   TROMZO_EXTERNAL_ID=<EXTERNAL_ID_FROM_TROMZO>
   aws cloudformation create-stack-set \
       --stack-set-name tromzo-iam-integration \
       --description "CloudFormation Template for Tromzo AWS Integration" \
       --capabilities CAPABILITY_NAMED_IAM \
       --template-body file://tromzo_integration_iam_role.json \
       --permission-model SERVICE_MANAGED \
       --auto-deployment '{"Enabled":false}' \
       --parameters \
       ParameterKey=TromzoAccountId,ParameterValue=$TROMZO_ACCOUNT_ID \
       ParameterKey=TromzoExternalId,ParameterValue=$TROMZO_EXTERNAL_ID
   ```

4. Create the stack instances in the given organizational units. You can check
   the organization root ID and manage the organizational units at
   [AWS Organizations]. Also, please specify exactly one region since the IAM
   role will be created globally.

   [AWS Organizations]: https://us-east-1.console.aws.amazon.com/organizations/v2/home/accounts

   ```sh
   ORG_UNIT_ID=<ROOT_ORGANIZATION_OR_ORGANIZATIONAL_UNIT>
   REGION=<YOUR_REGION>
   aws cloudformation create-stack-instances \
       --stack-set-name tromzo-iam-integration \
       --deployment-targets "{\"OrganizationalUnitIds\":[\"$ORG_UNIT_ID\"]}" \
       --regions "[\"$REGION\"]" \
       --operation-preferences FailureToleranceCount=0,MaxConcurrentCount=1
   ```

   Copy the `OperationId` from the output.  

5. Verify that the StackSet has completed successfully by running the following
   command. Note that it might take some time.

   ```sh
   aws cloudformation list-stack-set-operation-results \
       --stack-set-name tromzo-iam-integration \
       --operation-id $OPERATION_ID_FROM_CREATE_STACK_INSTANCES
   ```

   Copy the output.

   _Note: If you didn't get the `OperationId` from previous step, you can list
   the operations with the following command:_

   ```sh
   aws cloudformation list-stack-set-operations \
       --stack-set-name tromzo-iam-integration
   ```

To complete the integration, provide the output of the 
`list-stack-set-operation-results` command from step 5 to us.

## Steps to configure the AWS integration in On-Prem

1. Download the CloudFormation JSON from the following link
   https://github.com/tromzo/aws-integration/blob/main/tromzo_integration_onprem.json 

2. Execute the following script:

   ```sh
   aws cloudformation create-stack \
       --stack-name tromzo-iam-integration \
       --capabilities CAPABILITY_NAMED_IAM \
       --template-body file://tromzo_integration_onprem.json
   ```
3. Verify that the stack creation has completed successfully by running the following
   command:

   ```sh
   aws iam get-role --role-name TromzoSecurityAuditRole
   ```
   
   The output will contain the role ARN, which you'll need later to configure the integration in Tromzo.
   It looks like this: `arn:aws:iam::<YOUR_ACCOUNT_ID>:role/TromzoSecurityAuditRole`.

4. Create access keys for the security audit user:

   ```sh
   aws iam create-access-key --user-name tromzo-security-audit-user
   ```
   The output will contain `AccessKeyId` and `SecretAccessKey`, which are also needed to configure the integration.

After completing the steps above, you can create an integration in the app by specifying Role ARN, Access Key Id, 
and Secret Access Key in the appropriate fields in Tromzo UI.

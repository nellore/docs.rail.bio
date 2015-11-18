We assume that the administrator and the new user have both installed the AWS [Command Line Interface (CLI)](https://aws.amazon.com/cli/).  This is installed automatically with Rail-RNA, but can also be installed seprately.

* Understand best practices
    * The [National Institutes of Health (NIH)](http://www.nih.gov) and [Amazon Web Services (AWS)](https://aws.amazon.com) have published guidance on how to analyze protected genomic data, including [dbGaP](http://www.ncbi.nlm.nih.gov/gap)-protected data, securely on AWS.
        * [NIH Genomic Data Sharing (GDC)](https://gds.nih.gov)
        * [AWS Security Best Practices](https://aws.amazon.com/whitepapers/aws-security-best-practices/)
        * [Achirecting for Genomic Data Security and Compliance in AWS](https://d0.awsstatic.com/whitepapers/compliance/AWS_dBGaP_Genomics_on_AWS_Best_Practices.pdf) (If this document is out of date, look for latest published document on the [AWS Compliance](https://aws.amazon.com/compliance/compliance-latest-news/) page)

* Create the account
    * Navigate to http://aws.amazon.com/free in your web browser
    * Click "Get started today"
    * Check the "I am a new user" box and and continue to follow the instructions to create your new account.  You will enter, for example, contact and payment information.  Note that the free "Basic" level of service is sufficient for using Rail-RNA.

* Make a note of your account number
    * Log into the [AWS console] using the new account's email address and password
    * Click on the arrow next to your user name in the gray banner at the top of the page.  Select "My Account"
    * The account number is displayed at the top of the page

* Secure the account
    * Log into the [AWS console] using the new account's email address and password
    * Open the "Identity and Access Management" page
    * Under "security status" click "Activate MFA on your root account" and follow the instructions to enable multi-factor authentication
    * Under "Apply an IAM password policy" click "Manage password policy."  Configure the password policy according to the requirements mentioned in the "NIH Security Best Practices for Controlled-Access Data Subject to the NIH Genomic Data Sharing (GDS) Policy."  This usually entails the following, but please note that your institution might improse more stringent requirements:
        * Requiring a minimum password length of 12
        * Requiring at least one uppercase letter
        * Requiring at least one lowercase letter
        * Requiring at least one number
        * Requiring at least one non-alphanumeric character
        * Enable password expiration after 120 days
    * Click "Apply password policy"

* Create a user (administrator & user)
    * For this process, it is best for the account administrator must sit physically with the new user in order to minimize passing of credentials.
    * From the new user's laptop, the administrator logs into the [AWS Console] and and selects "Identity and Access Management"
    * The adminisrtator clicks "Create new users" and enters the new users username.  Check the "create an access key" checkbox.  Click "Create account"
    * Download and configure the user's AWS credentials.
        * The administrator or the user downloads the AWS credentials (including the AWS Access Key ID and AWS Secret Access Key) to the user's computer.  The user must immediately make the file containing the credentials readable only by the user.  The user must never share these credentials, intentionally or inadvertently, with anyone else.
        * The user can then register these credentials with the AWS CLI by entering `aws configure --profile dbgap`.  The user then enters the AWS Access Key ID, AWS Secret Access Key, and a default region (typically us-east-1).  A default output format need not be specified.
        * At this point, the new user can issue AWS API calls via the AWS command-line tools.  The credentials have been registered with the CLI and the original credentials file downloaded from the AWS Console may not be deleted.
    * Set user's password
        * Administrator: return to the [AWS Console] and select "Identity and Access Management"
        * Select "Users" on the left sidebar
        * Select the new user
        * Under "User Actions" menu select "Manage Password"
        * Select auto-generated password and check the box labeled "require user to create a new password at next sign-in"
        * Click "Apply"
        * Download the credentials to the user's computer.  This credentials file contains the username, the auto-generated password, and the URL for the account-specific login page.
        * The user should navigate to the login page URL, log in, and change the password.
        * At this point, the new user can log into the AWS console.

* Create the secure dbGaP CloudFormation stack with the template from the Rail-RNA repository
    * The user will find the appropriate CloudFormation template at `$RAILDOTBIO/cloudformation/dbgap.template` and can send it to the administrator, but it is recommended that the administrator grab the latest version of the template [here](https://raw.githubusercontent.com/nellore/rail/master/src/cloudformation/dbgap.template).
    * Click CloudFormation in the AWS console, making sure the region in the upper-right corner of the screen is the same as the user's default region (typically us-east-1, or N. Virginia).
    * Click "Create Stack".
    * Under choose a template, opt to upload `dbgap.template` to Amazon S3, and click "Next".
    * Under "Stack name", write "dbgap".
    * Under "Parameters", let the user select the name of a secure bucket into which they will write all of Rail-RNA's output. The bucket name should not have been taken by any other S3 user.
    * Click "Next" and "Next" again, then click "Create" and wait for the stack creation to complete. (The status message "CREATE_COMPLETE" will appear next to dbgap on the list of stacks.)

* Delegate Elastic MapReduce and CloudFormation authorites to user. The policies attached to the user must include iam:GetRole, iam:PassRole, and cloudformation:DescribeStacks. (administrator)
    * Click IAM in the AWS Console.
    * Click Policies, click Create Policy, then select Create Your Own Policy.
    * Under Policy Name, enter "UseExistingEMRRoles"
    * Under Policy Document, paste the following.
        ```
        {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:GetInstanceProfile",
                "iam:GetRole",
                "iam:PassRole"
            ],
            "Resource": "*"
        }
    ]
}
```
    * Click Create Policy
    * Click Users, and select the right user.
    * Click the Permissions tab.
    * Click Attach Policy, and select the AWSCloudFormationReadOnlyAccess, AmazonElasticMapReduceFullAccess, and UseExistingEMRRoles policies. Then click Attach Policy. Different policies including only some of the permissions from these may be included, but note that the user must be able to:
       (a) launch Elastic MapReduce clusters into the VPC from the secure dbGaP CloudFormation stack created by the administrator above, and
       (b) read and write to the secure S3 bucket created by the administrator on behalf of the user.

* Create the default EMR roles (administrator & user)
    * Follow [these instructions](http://docs.aws.amazon.com/ElasticMapReduce/latest/DeveloperGuide/emr-iam-roles-creatingroles.html) to create default roles for Elastic MapReduce.
    * The user should run `aws emr create-default-roles --profile dbgap` to retrieve the default Elastic MapReduce roles created by the administrator.

As for any new AWS account, you should consider how you would like to configure billing.  [Consolidated billing](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/consolidated-billing.html) can be convenient if you are managing multiple AWS accounts simultaneously.

Also, you should consider raising your EC2 instance limits.  This is particularly crucial if you plan to analyze large datasets (more than 100 RNA-seq samples at a time).  To raise your limits, navigate to the [AWS console], sign in, select EC2, select "limits" in the left sidebar, then click "Request limit increase" next to the relevant limits and follow the instructions.  We typically recommend that Rail-RNA users use c3.2xlarge or c3.8xlarge instances.

[AWS console]: https://aws.amazon.com/console/

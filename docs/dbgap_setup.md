* Understand best practices
    * The National Institutes of Health (NIH) and Amazon Web Services (AWS) both have published documents that will help you understand how to analyze protected genomic data securely.
        * NIH Genomic Data Sharing (GDC): https://gds.nih.gov
        * AWS Security Best Practices: https://aws.amazon.com/whitepapers/aws-security-best-practices/
        * Achirecting for Genomic Data Security and Compliance in AWS: https://d0.awsstatic.com/whitepapers/compliance/AWS_dBGaP_Genomics_on_AWS_Best_Practices.pdf (If this document is out of date, look for latest published document on the AWS Compliance page: https://aws.amazon.com/compliance/compliance-latest-news/)

* Create the account
    * Navigate to http://aws.amazon.com/free in your web browser
    * Click "Get started today"
    * Check the "I am a new user" box and and continue to follow the instructions to create your new account.  You will enter, for example, contact and payment information.  Note that the free "Basic" level of service is sufficient for using Rail-RNA.

* Make a note of your account number
    * Make a note of your account number

* Secure the account
    * Once the account is created, log into the AWS console using the new account's email address and password: https://aws.amazon.com/console/
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

* Create a user
    * For this process, it is best to have the new user physically with you in order to minimize passing of credentials.
    * 

As for any new AWS account, you should consider how you would like to configure billing.  [Consolidated billing](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/consolidated-billing.html) can be convenient if you are managing multiple AWS accounts simultaneously.

Also, you should consider raising your EC2 instance limits.  This is particularly crucial if you plan to analyze large datasets (more than 100 RNA-seq samples at a time).  To raise your limits, navigate to the AWS console (https://aws.amazon.com/console/), sign in, select EC2, select "limits" in the left sidebar, then click "Request limit increase" next to the relevant limits and follow the instructions.  We typically recommend that Rail-RNA users use c3.2xlarge or c3.8xlarge instances.

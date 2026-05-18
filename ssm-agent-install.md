Part 1: AWS Console Configuration (IAM & EC2)Step 1: Create the IAM RoleAn EC2 instance requires explicit permissions to securely communicate with the AWS Systems Manager platform.Open the AWS IAM Console (://amazon.com).Click Roles in the left sidebar, then click Create role.For Trusted entity type, select AWS service.For Service or use case, select EC2 from the dropdown menu, then click Next.In the permissions search box, type exactly: AmazonSSMManagedInstanceCore.Check the selection box next to AmazonSSMManagedInstanceCore, then click Next.Name the role something clear (e.g., ssm-test), and click Create role.Step 2: Attach the Role to Your EC2 InstanceThis links the security profile container directly to your virtual machine hardware.Open the Amazon EC2 Console (://amazon.com).Click Instances in the left sidebar and select your target instance.Click the Actions menu at the top right, then choose Security > Modify IAM role.Select the role you created (ssm-test) from the dropdown menu.Click Update IAM role to apply it.Part 2: Operating System ConfigurationBecause the AWS metadata service and the SSM Agent aggressively cache identity tokens, you must purge the system host memory paths to ensure the instance does not get stuck attempting to authenticate with old, unprivileged credentials.Log into your instance terminal via SSH and execute the following commands in order:bash# 1. Install the latest production package for enterprise Linux architectures
sudo yum install -y https://amazonaws.com

# 2. Stop the agent to safely clear internal data
sudo systemctl stop amazon-ssm-agent

# 3. Wipe the local credential cache (prevents the AccessDenied loop)
sudo rm -rf /var/lib/amazon/ssm/vault/
sudo rm -rf /var/lib/amazon/ssm/registration/

# 4. Configure the service to start automatically whenever the server boots
sudo systemctl enable amazon-ssm-agent

# 5. Start the SSM agent service immediately
sudo systemctl start amazon-ssm-agent
Use code with caution.Part 3: Operational VerificationStep 4: Verify SuccessConfirm that everything is linked correctly by running this final verification log check command:bashsudo tail -n 20 /var/log/amazon/ssm/amazon-ssm-agent.log
Use code with caution.Verification Success Indicator: Look for log output stating INFO [amazon-ssm-agent] Entering online loop or Connection to Systems Manager established successfully. This confirms your EC2 instance is completely configured, online, and ready.
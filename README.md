# Game server automation scripts
Scripts for game server automation

Supported games:

* Factorio 
* Valheim 

## Game Server
So you want a quick and simple way to create your own game server?

This repo has Cloudformation Templates (CFTs) that can be used with Amazon Web Services to automatically provision a game server with little effort.

### New to AWS?

[AWS Website](https://aws.amazon.com/)
Create an account. Make sure to set up 2 factor authentication. [Guide here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html) 

Never share ssh keys, passwords, aws access keys or aws console access. 

If this is your first experience with AWS, I'd also recommend setting up a billing alarm. [Guide Here](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html)

## Using the included cloud formation templates

Follow these steps to create a game server.

### Choose a region

AWS has locations scattered across the world. Choose the region closest to your location for the lowest latency. [List can be found here.](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)

To change your region log into the [AWS console](https://console.aws.amazon.com/console/home), and select the region from the drop down in the top right corner. 

### Create an EC2 SSH keypair

Creating a key pair is required to be able to access your server later via ssh.

1. Navigate to the EC2 service page.
2. [In the menu, under 'Network and Security', select Key Pairs.](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#KeyPairs:)
3. In the top right corner, click on 'Create key pair'. 
4. Name the keypair your desired name. For Valheim, I would recommend 'valheim-ssh', for Factorio 'factorio-ssh'. The CFTs use this when setting up the server.
   Select .pem if you plan on using ssh, or .ppk if you plan on using Putty.
5. Create the key, the .pem or .ppk file will be downloaded.
6. If you set up a custom name, you will need to change the name of the key referenced in the CFT you use.

### Creating the Cloudformation stack.

1. [Navigate to the Cloudformation service page.](https://console.aws.amazon.com/cloudformation)
2. Click Create Stack > Upload a template file > Choose file
3. Select the template from this repo you want to use. `*.cft.yaml`
4. Enter any name
5. Select an instance type (defaults to t3.medium). You can change this later
6. Select the EC2 keypair you just created. 
7. Click 'Next' > Scroll Down, Click 'Next' > Select 'I acknowledge that AWS CloudFormation might create IAM resources.'
8. Click 'Create Stack'

You can watch the resources being made. This spins up the EC2 server that runs the game, creates an S3 bucket where save files will be backed up to, and creates security (IAM) policies to manage those. 

### Server information

To get the server's IP address

1. [Navigate to the EC2 dashboard](https://console.aws.amazon.com/ec2/v2/home)
2. Click on 'Running instances'
3. Select your instance.
4. IP address is under 'Public IPv4 address'. 

### Managing your server

[Guide to log into your EC2 instance](https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-2-connect-to-instance.html)

The Valheim and Palworld CFT uses LGSM to manage the server. [Documentation here](https://linuxgsm.com/lgsm/vhserver/)
``` sh
# You can log in as the Valheim server user via:
sudo su
su - vhserver # replace vhserver with pwserver for palworld
# Example command
./vhserver monitor
# config is stored here: /home/vhserver/lgsm/config-lgsm/vhserver/common.cfg

```

We created the factorio CFT prior knowing of LGSM integration, so it's a systemctl process. We'd like to migrate this eventually

``` sh
# Example commands
systemctl start factorio.service
systemctl stop factorio.service
systemctl restart factorio.service
```

## FAQ

### What size server should I use?

A t3.medium runs both servers pretty well. You can always upgrade later without recreating everything.

### What's in the 'python' folder?

This folder contains python snippets that can query and alter the state of the EC2 instances in your account. I have these in lambda fuctions that are triggers via api gateway, allowing me programatic access to change instance state of my EC2 instances.
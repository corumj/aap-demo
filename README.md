# Ansible Automation Platform Demo 
This repo of playbooks is designed to help Red Hatters easily deploy an AAP 2.x environment on the RHPDS AWS Blank Open Environment.  It might work with a non RHPDS AWS environment provided you are using Route53 to manage your DNS.  I don't provide support for this repository, so use at your own risk.  

## Requirements 
1. Ansible-core installed locally 
2. Use your AWS Blank Open Environment credentials to configure the CLI.  Directions here: [AWS Credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
  
    TLDR; I expect credentials in the following format at ~/.aws/credentials 
    ```ini 
    [default]
    region = us-east-2
    aws_access_key_id = ** <access key> **
    aws_secret_access_key = ** <secret key> **
    ```
3. Employee Subscriptions signed up for and Simple Content Access enabled.
4. Create an [ansible.cfg](https://docs.ansible.com/ansible/latest/reference_appendices/config.html) file in the root of this repo (or use a global one) with correct tokens to consume Automation Hub content  
    ```ini
    [defaults]
    host_key_checking = False
    # callback_whitelist = profile_tasks

    [galaxy]
    server_list = automation_hub, galaxy 

    [galaxy_server.automation_hub]
    url=https://console.redhat.com/api/automation-hub/
    auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
    token= ** <automation hub token> **

    [galaxy_server.galaxy]
    url=https://galaxy.ansible.com
    ```
5. Create an extra_vars.yml file in the root of this repository.  It is specifically in the .gitignore along with the above config files for a reason, don't check in your secret stuff.  The extra_vars.yml file should have this content:
    ```yaml
    --- 
    # These variables may change each time you spin up this environment
    # Don't forget to update your `~/.aws/credentials` file
    top_level_domain: < Open Environment Email provides this, ex: sandbox123.opentlc.com >

    # These variables will require a one-time setup 

    # Begin Let's Encrypt 
    certificate_env: staging # staging or prod, only switch to prod when you've confirmed your setup works
    cert_email: < email you want cert expiry notifications to go to> 
    certificate_force: false # if you have issues with SSL certs set to true

    # End Let's Encrypt  

    # Begin AWS 
    aws_profile: default 
    aws_region: us-east-2 

    # End AWS 

    # Begin General 
    # Required for podman authentication to registry.redhat.io
    redhat_username: < red hat username >
    redhat_password: < red hat password >

    # default admin password for all web/api consoles and windows logins 
    admin_password: < initial admin password for AAP itself > 

    # default ansible user (changes for Windows)
    ansible_user: ec2-user 

    # Key pair 
    public_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"  
    private_key: "{{ lookup('file', '~/.ssh/id_rsa') }}"  

    # End General 
    ```
6. Download the AAP Installer, name it `aap.tar.gz` and save it in this repo 
7. Create a Manifest to license AAP and name it `manifest.zip`, place it in this repo

## To Use
It is vitally important that you pay attention and shut these instances down, they're quite large and thus somewhat expensive.  
1. To start setup, after the above is complete run `ansible-playbook -i aws_ec2.yml -e @extra_vars.yml controller_setup.yml` 
2. To teardown the environment, run `ansible-playbook -i aws_ec2.yml -e @extra_vars.yml controller_teardown.yml`

So long as you keep the `generated_files` directory and have the same `top_level_domain` you can teardown and spin up as many times as you want until your AWS OpenEnvironment expires or you remove it from RHPDS.  I commonly teardown every night, starting up if I need the environment (I think it takes around an hour, but I've honestly never timed it).

## Todo 
1. Add option to include demo playbooks using redhat_cop.controller_configuration
2. Test if Shutting off instances and restarting them works with SSO and Catalog in the mix, this is because it's a lot faster to shut down and restart then reprovision and reinstall.
3. Test if teardown playbook works if you've provisioned other crap in the environment 
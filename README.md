

Update your [AWS Credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
I expect credentials in the following format at ~/.aws/credentials 
``` 
[default]
region = us-east-2
aws_access_key_id = <access key>
aws_secret_access_key = <secret key>
```

Create an [ansible.cfg](https://docs.ansible.com/ansible/latest/reference_appendices/config.html) file in the root of this repo (or use a global one) with correct tokens to consume Automation Hub content  
```
[defaults]
host_key_checking = False
# callback_whitelist = profile_tasks

[galaxy]
server_list = automation_hub, galaxy 

[galaxy_server.automation_hub]
url=https://console.redhat.com/api/automation-hub/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
token=<automation hub token>

[galaxy_server.galaxy]
url=https://galaxy.ansible.com
```
Must have Simple Content Access active

---
title: AWS CLI
author: Peng-Yu Chen
date: 2023-04-02
tags:
  - AWS
---

> Learn how to set up AWS CLI for your AWS account in a few easy steps! This
> comprehensive guide walks you through enabling IAM Identity Center, adding a
> user to your AWS account, installing and configuring AWS CLI V2, and setting
> AWS environment variables. You'll even get to test your AWS CLI skills by
> creating a role using AWS CLI V2. Follow along and become an AWS CLI pro!

## Enable IAM identity center

1. Sign in as root user
1. Go to
   [IAM Identity Center console](https://console.aws.amazon.com/singlesignon).
1. Under **Enable IAM Identity Center**, click **Enable**.
1. Click **Create AWS organization** in the popup window.
   ![](https://i.imgur.com/q7Y2Cnh.png)

## Add User in IAM Identity Center

1. Click **Add user** in "IAM Identity Center > Users".
1. Fill the required information. ![](https://i.imgur.com/wkIaRJg.png)
1. Review the information and click **Add user** in the bottom right. You can
   also create a group for this user if you like.
   ![](https://i.imgur.com/5qLbS9S.png)
1. Hooray! The user "buildwebapp2023" was successfully added!
   ![](https://i.imgur.com/gLK97Op.png)
1. Finally, navigate to your mail and click Accept invitation.
   ![](https://i.imgur.com/krJQ7Ph.png)

## Add User to an AWS account

1. In "IAM Identity Center > AWS Organizations: AWS accounts", click one of the
   organization then click **Assign users or groups**.
1. Select the newly added user and "AdministratorAccess" permission sets (We'll
   need to create a role programmatically it later).
1. Review and click **Submit**. Wait a second...
1. The user with selected permission sets is assigned to this AWS account!

## Install and Configure AWS CLI V2

1. Install AWS CLI via Homebrew

   ```bash
   brew install awscli
   ```

1. Specify an alternate location to store AWS config and credentials following
   the
   [XDG Base Directory](https://wiki.archlinux.org/title/XDG_Base_Directory). I
   use zsh, so I'll do

   ```bash
   echo 'export AWS_CONFIG_FILE=$HOME/.config/aws/config
   export AWS_SHARED_CREDENTIALS_FILE=$HOME/.config/aws/credentials' > ~/.config/zsh/init/aws.zshrc
   ```

1. Generate the config file for `sso-session` and `profile` by grabbing the
   information in "IAM Identity Center > Dashboard", clicking the AWS access
   portal URL under **Settings summary**, and copying and pasting the **SSO
   Start URL** and **SSO Region** to proper fields. Alternatively, use the
   `aws configure sso` wizard.

   ```bash
   AWS_ACCOUNT_ID=123456789012
   echo '[sso-session my-sso]
   sso_start_url = https://d-9067911059.awsapps.com/start#
   sso_region = us-east-1
   sso_registration_scopes = sso:account:access

   [profile admin-access]
   sso_session = my-sso
   sso_account_id = '${AWS_ACCOUNT_ID}'
   sso_role_name = AdministratorAccess
   region = us-east-1
   output = yaml' > ~/.config/aws/config
   ```

## Set AWS environment variables

To set AWS environment variables, copy the export statements from the same place
as above.

```bash
echo 'export AWS_ACCESS_KEY_ID="XXX"
export AWS_SECRET_ACCESS_KEY="YYY"
export AWS_SESSION_TOKEN="ZZZ"' >> ~/.config/zsh/init/aws.zshrc
```

## Test your AWS CLI

To test your AWS CLI, you can create a role and attach a policy for it
programmatically. The command is:

```bash
AWS_ACCOUNT_ID=123456789012
echo '{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::'${AWS_ACCOUNT_ID}':oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:walkccc/go-boilerplate:*"
        }
      }
    }
  ]
}' > GitHubActionsRole.json
aws iam create-role --role-name GitHubActionsRole --assume-role-policy-document file://GitHubActionsRole.json
aws iam attach-role-policy --role-name GitHubActionsRole --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryPowerUser
rm GitHubActionsRole.json
```

Congrats, you've succesfully created a role via AWS CLI V2! 🙂

Finally, when you're done, clean up the resources you created:

```bash
aws iam detach-role-policy --role-name GitHubActionsRole --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryPowerUser
aws iam delete-role --role-name GitHubActionsRole
```

## References

- [Step 1: Enable IAM Identity Center - AWS IAM Identity Center (successor to AWS Single Sign-On)](https://docs.aws.amazon.com/singlesignon/latest/userguide/get-started-enable-identity-center.html?icmpid=docs_sso_console)
- [Token provider configuration with automatic authentication refresh for AWS IAM Identity Center (successor to AWS Single Sign-On)](https://docs.aws.amazon.com/cli/latest/userguide/sso-configure-profile-token.html)

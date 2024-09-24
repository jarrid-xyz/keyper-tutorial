# 2. Create App Configuration and Credentials (AWS)

Now, let's create the necessary application configuration and credentials for [Keyper](https://jarrid.xyz/keyper).

## Application Configuration

[Keyper](https://jarrid.xyz/keyper) provides a simple interface for modifying resource creation and provisioning behavior using [cdktf](https://developer.hashicorp.com/terraform/cdktf/cli-reference/commands). For more details, go to [Keyper Deployment AWS](https://jarrid.xyz/keyper/deploy/aws/). Here, we will cover the required configurations to point Keyper to your cloud provider.

### Provider

Provider configuration is used by the deploy module.

#### `tfcdk`

Specify [cdktf](https://developer.hashicorp.com/terraform/cdktf/cli-reference/commands) execution configurations.

- `stack`: specify the cloud provider to be `aws`. ***Note: defaults to `gcp`.***

#### Cloud Provider

You can configure cloud providers such as `gcp` or `aws` for resource provisioning. For this tutorial, we will be using `aws`. You can read more on the [Keyper Deployment Setup AWS page](https://jarrid.xyz/keyper/deploy/aws/).

Here's an example of the cloud provider configuration in `app.yaml`:

```yaml {"id":"01J72T8D99G7BYKPEGCXP8Z1ZK"}
provider:
  tfcdk:
    stack: "aws"
  aws:
    region: us-east-1
    backend:
      type: cloud
    assume_role_arn: <assume role arn>
```

- `region`: You can specify where you'd like to deploy your resources to, check out the list of available AWS regions and zones [here](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/).
- `assume_role_arn`: [Terraform](https://www.terraform.io/) will need to use a pre-configured AWS role for resource deployment and remote state backend. See [Keyper Deployment Setup AWS](https://jarrid.xyz/keyper/deploy/aws/#create-resource-admin-iam-role) for more details.

##### Create AWS Role

First, let's install aws cli. If you are running this on your local machine, follow [AWS's guide to install aws cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

```bash {"id":"01J72T8D99G7BYKPEGCY2D7XP5"}
curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Once the aws cli is installed, log in to your AWS account.

```bash {"id":"01J72TQ6GXPEDZ6119DEECFXP3"}
aws configure
```

Next, let's create a role to run [Terraform](https://www.terraform.io/) for this tutorial.

```bash {"id":"01J79MFQK0VDDJMWNH30WF3TGE"}
tee assume-role-policy.json <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<ACCOUNT_ID>:root"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "ArnLike": {
                    "aws:PrincipalArn": "arn:aws:iam::<ACCOUNT_ID>:user/<USER>"
                }
            }
        }
    ]
}
EOF
```

Replace `<ACCOUNT_ID>` with your own aws account id and `<USER>` with the user you are using in your aws cli config. __Note: this setup is for demo purpose. Your company or organization should have default CI/CD setup.__

```bash {"id":"01J79KPX61821053RVJQ939PBH"}
SERVICE=keyper
aws iam create-role \
    --role-name $SERVICE-cdktf-role \
    --assume-role-policy-document file://assume-role-policy.json
```

Next, let's add KMS and IAM admin permission to the role so it has the permission to create/destroy service accounts and keys.

```bash {"id":"01J79Q90W3NFS1B319T4TG3KKZ"}
tee kms-policy.json <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kms:*"
            ],
            "Resource": "*"
        }
    ]
}
EOF
```

```bash {"id":"01J79Q90W3NFS1B319T6CP4G5W"}
SERVICE=keyper
aws iam put-role-policy \
    --role-name $SERVICE-cdktf-role \
    --policy-name $SERVICE-cdktf-kms-policy \
    --policy-document file://kms-policy.json
```

```bash {"id":"01J79QBPMYM3NE0Z5RJE2VWGN9"}
SERVICE=keyper
aws iam attach-role-policy \
    --role-name $SERVICE-cdktf-role \
    --policy-arn arn:aws:iam::aws:policy/IAMFullAccess
```

In this tutorial, as this container is shortlived, we will be using remote backend. We add S3 permission to the role so that the role can manage [Terraform's remote backend](https://developer.hashicorp.com/terraform/language/settings/backends/remote).

```bash {"id":"01J79QS1NFH8713YVK7E9BM0BY"}
tee s3-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::keyper-tf-state"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::keyper-tf-state/*"
    }
  ]
}
EOF
```

```bash {"id":"01J79QS1NFH8713YVK7GT6B4PA"}
SERVICE=keyper
aws iam put-role-policy \
    --role-name $SERVICE-cdktf-role \
    --policy-name $SERVICE-cdktf-backend-policy \
    --policy-document file://s3-policy.json
```

Now, we are ready to create the configuration override in `app.local.yaml`. Open the `app.local.yaml` file:

```bash {"cwd":"../keyper","id":"01J79KPX61821053RVJS4VCWZJ"}
touch ./app.local.yaml
code ./app.local.yaml
```

Add the following to the `app.local.yaml`. __Remember to update `<assume_role_arn>` accordingly.__

```yaml {"id":"01J4JN2QZQJ63YME8GVHZH2XQ1"}
provider:
  tfcdk:
    stack: "aws"
  aws:
    region: us-east-1
    backend:
      type: cloud
    assume_role_arn: <assume_role_arn>
resource:
  backend:
    backend: local
    path: configs
out_dir: "/home/keyper"
```

### Resource

Resource configuration is used by the resource module. In [Keyper](https://jarrid.xyz/keyper), resource configuration are separated generated in Json so it's easy for user to understand, track and configure if necessary.

In the configuration we added, we set configuration to be stored locally (currently we only support local) and in configs directory.

```yaml {"id":"01J79KPX61821053RVJSE6EJ5R"}
resource:
  backend:
    backend: local
    path: configs
```

Now, let’s go to the next tutorial: ➡️ [Create Deployment, Role and Key](../3-create-deployment-role-and-key/README.md)

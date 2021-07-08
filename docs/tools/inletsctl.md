# inletsctl reference documentation

inletsctl is an automation tool for inlets/-pro.

Features:
* `create` / `delete` cloud VMs with inlets/-pro pre-installed via systemd
* `download [--pro]` - download the inlets/-pro binaries to your local computer
* `kfwd` - forward services from a Kubernetes cluster to your local machine using inlets/-pro

View the code on GitHub: [inlets/inletsctl](https://github.com/inlets/inletsctl)

## Install `inletsctl`

You can install inletsctl using its installer, or from [the GitHub releases page](https://github.com/inlets/inletsctl/releases/)

```bash
# Install to local directory (and for Windows users)
curl -sLSf https://inletsctl.inlets.dev | sh

# Install directly to /usr/local/bin/
curl -sLSf https://inletsctl.inlets.dev | sudo sh
```

Windows users are encouraged to use [git bash](https://git-scm.com/downloads) to install the inletsctl binary.

## Downloading inlets-pro

The `inletsctl download` command can be used to download the inlets/-pro binaries.

Example usage:

```sh
# Download the latest inlets-pro binary
inletsctl download --pro

# Download a specific version of inlets-pro
inletsctl download --pro --version 0.8.5
```

## The `create` command

### Create a HTTPS tunnel with a custom domain

This example uses DigitalOcean to create a cloud VM and then exposes a local service via the newly created exit-server.

Let's say we want to expose a Grafana server on our internal network to the Internet via [Let's Encrypt](https://letsencrypt.org/) and HTTPS?

```bash
export DOMAIN="grafana.example.com"

inletsctl create \
  --provider digitalocean \
  --region="lon1" \
  --access-token-file $HOME/do-access-token \
  --letsencrypt-domain $DOMAIN \
  --letsencrypt-email webmaster@$DOMAIN \
  --letsencrypt-issuer prod
```

You can also use `--letsencrypt-issuer` with the `staging` value whilst testing since Let's Encrypt rate-limits how many certificates you can obtain within a week.

Create a DNS A record for the IP address so that `grafana.example.com` for instance resolves to that IP. For instance you could run:

```bash
doctl compute domain create \
  --ip-address 46.101.60.161 grafana.example.com
```

Now run the command that you were given, and if you wish, change the upstream to point to the domain explicitly:

```bash
# Obtain a license at https://inlets.dev
# Store it at $HOME/.inlets/LICENSE or use --help for more options

# Where to route traffic from the inlets server
export UPSTREAM="grafana.example.com=http://192.168.0.100:3000"

inlets-pro http client --url "wss://46.101.60.161:8123" \
--token "lRdRELPrkhA0kxwY0eWoaviWvOoYG0tj212d7Ff0zEVgpnAfh5WjygUVVcZ8xJRJ" \
--upstream $UPSTREAM

To delete:
  inletsctl delete --provider digitalocean --id "248562460"
```

You can also specify more than one domain and upstream for the same tunnel, so you could expose OpenFaaS and Prometheus separately for instance.

Update the `inletsctl create` command with multiple domains such as: `--letsencrypt-domain openfaas.example.com --letsencrypt-domain grafana.example.com`

Then for the `inlets-pro client` command, update the upstream in the same way: `--upstream openfaas.example.com=http://127.0.0.1:8080,grafana.example.com=http://192.168.0.100:3000`

### Create a HTTP tunnel

This example uses Linode to create a cloud VM and then exposes a local service via the newly created exit-server.

```bash
export REGION="eu-west"

inletsctl create \
  --provider linode \
  --region="$REGION" \
  --access-token-file $HOME/do-access-token
```

You'll see the host being provisioned, it usually takes just a few seconds:

```
Using provider: linode
Requesting host: peaceful-lewin8 in eu-west, from linode
2021/06/01 15:56:03 Provisioning host with Linode
Host: 248561704, status: 
[1/500] Host: 248561704, status: new
...
[11/500] Host: 248561704, status: active

inlets PRO (0.7.0) exit-server summary:
  IP: 188.166.168.90
  Auth-token: dZTkeCNYgrTPvFGLifyVYW6mlP78ny3jhyKM1apDL5XjmHMLYY6MsX8S2aUoj8uI
```

Now run the command given to you, changing the `--upstream` URL to match a local URL such as `http://127.0.0.1:3000`

```bash
# Obtain a license at https://inlets.dev
export LICENSE="$HOME/.inlets/license"

# Give a single value or comma-separated
export PORTS="8000"

# Where to route traffic from the inlets server
export UPSTREAM="localhost"

inlets-pro client --url "wss://188.166.168.90:8123/connect" \
  --token "dZTkeCNYgrTPvFGLifyVYW6mlP78ny3jhyKM1apDL5XjmHMLYY6MsX8S2aUoj8uI" \
  --license-file "$LICENSE" \
  --upstream $UPSTREAM \
  --ports $PORTS
```

You can then access your local website via the Internet and the exit-server's IP at:

http://165.232.108.137

When you're done, you can delete the host using its ID or IP address:

```bash
  inletsctl delete --provider linode --id "248561704"
  inletsctl delete --provider linode --ip "188.166.168.90"
```

### Create a tunnel for a TCP service

This example is similar to the previous one, but also adds link-level encryption between your local service and the exit-server.

In addition, you can also expose pure TCP traffic such as SSH or Postgresql.

```sh
inletsctl create \
  --provider digitalocean \
  --access-token-file $HOME/do-access-token \
  --pro
```

Note the output:

```bash
inlets PRO (0.7.0) exit-server summary:
  IP: 142.93.34.79
  Auth-token: TUSQ3Dkr9QR1VdHM7go9cnTUouoJ7HVSdiLq49JVzY5MALaJUnlhSa8kimlLwBWb

Command:
  export LICENSE=""
  export PORTS="8000"
  export UPSTREAM="localhost"

  inlets-pro client --url "wss://142.93.34.79:8123/connect" \
        --token "TUSQ3Dkr9QR1VdHM7go9cnTUouoJ7HVSdiLq49JVzY5MALaJUnlhSa8kimlLwBWb" \
        --license "$LICENSE" \
        --upstream $UPSTREAM \
        --ports $PORTS

To Delete:
          inletsctl delete --provider digitalocean --id "205463570"
```

Run a local service that uses TCP such as MariaDB:

```bash
head -c 16 /dev/urandom |shasum 
8cb3efe58df984d3ab89bcf4566b31b49b2b79b9

export PASSWORD="8cb3efe58df984d3ab89bcf4566b31b49b2b79b9"

docker run --name mariadb \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=8cb3efe58df984d3ab89bcf4566b31b49b2b79b9 \
-ti mariadb:latest
```

Connect to the tunnel updating the ports to `3306`

```bash
export LICENSE="$(cat ~/LICENSE)"
export PORTS="3306"
export UPSTREAM="localhost"

inlets-pro client --url "wss://142.93.34.79:8123/connect" \
      --token "TUSQ3Dkr9QR1VdHM7go9cnTUouoJ7HVSdiLq49JVzY5MALaJUnlhSa8kimlLwBWb" \
      --license "$LICENSE" \
      --upstream $UPSTREAM \
      --ports $PORTS
```

Now connect to your MariaDB instance from its public IP address:

```bash
export PASSWORD="8cb3efe58df984d3ab89bcf4566b31b49b2b79b9"
export EXIT_IP="142.93.34.79"

docker run -it --rm mariadb:latest mysql -h $EXIT_IP -P 3306 -uroot -p$PASSWORD

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.5.5-MariaDB-1:10.5.5+maria~focal mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database test; 
Query OK, 1 row affected (0.039 sec)
```

## Examples for specific cloud providers

### Example usage with AWS EC2

To use the instructions below you must have the AWS CLI configured with sufficient permissions to 
create users and roles. 

* Create a AWS IAM Policy with the following:

Create a file named `policy.json` with the following content

```json 
{
    "Version": "2012-10-17",
    "Statement": [  
        {
            "Effect": "Allow",
            "Action": [
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:DescribeInstances",
                "ec2:DescribeImages",
                "ec2:TerminateInstances",
                "ec2:CreateSecurityGroup",
                "ec2:CreateTags",
                "ec2:DeleteSecurityGroup",
                "ec2:RunInstances",
                "ec2:DescribeInstanceStatus"
            ],
            "Resource": ["*"]
        }
    ]
}
```

Create the policy in AWS 

```sh 
aws iam create-policy --policy-name inlets-automation --policy-document file://policy.json
```


* Create an IAM user
```sh 
aws iam create-user --user-name inlets-automation
```

* Add the Policy to the IAM user

We need to use the policy arn generated above, it should have been printed to the console on success. It also follows the format below.

```sh 
export AWS_ACCOUNT_NUMBER="Your AWS Account Number"
aws iam attach-user-policy --user-name inlets-automation --policy-arn arn:aws:iam::${AWS_ACCOUNT_NUMBER}:policy/inlets-automation
```

* Generate an access key for your IAM User 

The below commands will create a set of credentials and save them into files for use later on.

> we are using [jq](https://stedolan.github.io/jq/) here. It can be installed using the link provided.
> Alternatively you can print ACCESS_KEY_JSON and create the files manually.

```sh 
ACCESS_KEY_JSON=$(aws iam create-access-key --user-name inlets-automation)
echo $ACCESS_KEY_JSON | jq -r .AccessKey.AccessKeyId > access-key.txt
echo $ACCESS_KEY_JSON | jq -r .AccessKey.SecretAccessKey > secret-key.txt
```

Create an exit-server:

```bash
inletsctl create \
  --provider ec2 \
  --region eu-west-1 \
  --access-token-file ./access-key.txt \
  --secret-key-file ./secret-key.txt
```

Delete an exit-server:

```bash
export IP=""

inletsctl create \
  --provider ec2 \
  --region eu-west-1 \
  --access-token-file ./access-key.txt \
  --secret-key-file ./secret-key.txt \
  --ip $IP
```

### Example usage with Google Compute Engine

* One time setup required for a service account key

> It is assumed that you have gcloud installed and configured on your machine.
If not, then follow the instructions [here](https://cloud.google.com/sdk/docs/quickstarts)

```sh
# Get current projectID
export PROJECTID=$(gcloud config get-value core/project 2>/dev/null)

# Create a service account
gcloud iam service-accounts create inlets \
--description "inlets-operator service account" \
--display-name "inlets"

# Get service account email
export SERVICEACCOUNT=$(gcloud iam service-accounts list | grep inlets | awk '{print $2}')

# Assign appropriate roles to inlets service account
gcloud projects add-iam-policy-binding $PROJECTID \
--member serviceAccount:$SERVICEACCOUNT \
--role roles/compute.admin

gcloud projects add-iam-policy-binding $PROJECTID \
--member serviceAccount:$SERVICEACCOUNT \
--role roles/iam.serviceAccountUser

# Create inlets service account key file
gcloud iam service-accounts keys create key.json \
--iam-account $SERVICEACCOUNT
```

* Run inlets OSS or inlets-pro

```sh
# Create a tunnel with inlets OSS
inletsctl create -p gce --project-id=$PROJECTID -f=key.json

## Create a TCP tunnel with inlets-pro
inletsctl create -p gce -p $PROJECTID -f=key.json

# Or specify any valid Google Cloud Zone optional zone, by default it get provisioned in us-central1-a
inletsctl create -p gce --project-id=$PROJECTID -f key.json --zone=us-central1-a
```

### Example usage with Azure

Prerequisites:

* You will need `az`. See [Install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)


Generate Azure auth file 
```sh
az ad sp create-for-rbac --sdk-auth > ~/Downloads/client_credentials.json
```

Create
```sh
inletsctl create --provider=azure --subscription-id=4d68ee0c-7079-48d2-b15c-f294f9b11a9e \
  --region=eastus --access-token-file=~/Downloads/client_credentials.json 
```

Delete
```sh
inletsctl delete --provider=azure --id inlets-clever-volhard8 \
  --subscription-id=4d68ee0c-7079-48d2-b15c-f294f9b11a9e \
  --region=eastus --access-token-file=~/Downloads/client_credentials.json
```


### Example usage with Hetzner

```sh
# Obtain the API token from Hetzner Cloud Console.
export TOKEN=""

inletsctl create --provider hetzner \
  --access-token $TOKEN \
  --region hel1
```

Available regions are `hel1` (Helsinki), `nur1` (Nuremberg), `fsn1` (Falkenstein).

### Example usage with Linode

Prerequisites:

* Prepare a Linode API Access Token. See [Create Linode API Access token](https://www.linode.com/docs/platform/api/getting-started-with-the-linode-api/#get-an-access-token)  


Create
```sh
inletsctl create --provider=linode --access-token=<API Access Token> --region=us-east
```

Delete
```sh
inletsctl delete --provider=linode --access-token=<API Access Token> --id <instance id>
```

### Example usage with Scaleway

```sh
# Obtain from your Scaleway dashboard:
export TOKEN=""
export SECRET_KEY=""
export ORG_ID=""

inletsctl create --provider scaleway \
  --access-token $TOKEN
  --secret-key $SECRET_KEY --organisation-id $ORG_ID
```

The region is hard-coded to France / Paris 1.

## The `delete` command

The delete command takes an id or IP address which are given to you at the end of the `inletsctl create` command. You'll also need to specify your cloud access token.

```bash
inletsctl delete \
  --provider digitalocean \
  --access-token-file ~/Downloads/do-access-token \
  --id 164857028 \
```

Or delete via IP:

```bash
inletsctl delete \
  --provider digitalocean \
  --access-token-file ~/Downloads/do-access-token \
  --ip 209.97.131.180 \
```

## kfwd - Kubernetes service forwarding

kfwd runs an inlets-pro server on your local computer, then deploys an inlets client in your Kubernetes cluster using a Pod. This enables your local computer to access services from within the cluster as if they were running on your laptop.

inlets PRO allows you to access any TCP service within the cluster, using an encrypted link:

Forward the `figlet` pod from `openfaas-fn` on port `8080`:

```bash
inletsctl kfwd \
  --pro \
  --license $(cat ~/LICENSE)
  --from figlet:8080 \
  --namespace openfaas-fn \
  --if 192.168.0.14
```

Note the `if` parameter is the IP address of your local computer, this must be reachable from the Kubernetes cluster.

Then access the service via `http://127.0.0.1:8080`.

### Example usage with Vultr

Prerequisites:
* Prepare a Vultr API Key via your [Members Area](https://my.vultr.com/settings/#settingsapi)

```
# Obtain your Vultr API Key
export VULTR_API_KEY=""

inletsctl create --provider vultr \
  --access-token $VULTR_API_KEY \
  --region LHR
```

## Troubleshooting

inletsctl provisions a host called an exit node or exit server using public cloud APIs. It then 
prints out a connection string.

Are you unable to connect your client to the exit server?

### Troubleshooting inlets PRO

If using auto-tls (the default), check that port 8123 is accessible. It should be serving a file with a self-signed certificate, run the following:

```bash
export IP=192.168.0.1
curl -k https://$IP:8123/.well-known/ca.crt
```

If you see connection refused, log in to the host over SSH and check the service via systemctl:

```bash
sudo systemctl status inlets-pro

# Check its logs
sudo journalctl -u inlets-pro
```

You can also check the configuration in `/etc/default/inlets-pro`, to make sure that an IP address and token are configured.

## Configuration using environment variables

You may want to set an environment variable that points at your `access-token-file` or `secret-key-file`

Inlets will look for the following:

```sh
# For providers that use --access-token-file
INLETS_ACCESS_TOKEN

# For providers that use --secret-key-file
INLETS_SECRET_KEY
```

With the correct one of these set you wont need to add the flag on every command execution. 

You can set the following syntax in your `bashrc` (or equivalent for your shell)

```sh
export INLETS_ACCESS_TOKEN=$(cat my-token.txt)

# or set the INLETS_SECRET_KEY for those providors that use this
export INLETS_SECRET_KEY=$(cat my-token.txt)
```

# Akaunting

## Installation

1. Run `git clone https://github.com/frankhinek/akaunting.git`.
1. Run `cp .env.example .env`.
1. Edit the `.env` file and enter your database, NGINX, and AWS secrets.
1. Run the following commands

        export $(grep -v '^#' .env | xargs) && \
        sed -i '' "s/__NGINX_SERVER_NAME__/${NGINX_SERVER_NAME}/g" nginx/conf.d/akaunting.conf && \
        sed -i '' "s/__NGINX_DNS_ZONE__/${NGINX_DNS_ZONE}/g" nginx/conf.d/akaunting.conf

1. Run `docker-compose build`.

## Configure AWS Route 53 for Certbot

1. Navigate to https://console.aws.amazon.com/iam/home#/users
1. Click **Add User**.
1. Enter `acme-domain-writer` for **User name**.
1. Place a check next to **Programmatic access**.
1. Click the **Next: Permissions** button.
1. Click **Attach existing policies directly**.
1. Click **Create policy**.
1. Click the **JSON** tab and enter the following policy, changing the `HOSTED_ZONE_ID` value:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:ListHostedZones",
                "route53:GetChange"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "route53:ChangeResourceRecordSets"
            ],
            "Resource": [
                "arn:aws:route53:::hostedzone/HOSTED_ZONE_ID"
            ]
        }
    ]
}
```

1. Click **Next** and enter a name of `certbot-dns-route53-allow-writing-YOUR.DOMAIN-zone`.
1. Go back to the Add user browser tab, search for "certbot", and place a check next to the newly created policy.
1. Click the **Next: Tags** button.
1. Click the **Next: Review** button.
1. Click the **Create user** button.
1. Securely save the Access key ID and Secret access key
1. Click the **Close** button.

## Generate Let's Encrypt Certificates
Run the following to create the wildcard certificate, replacing the `DOMAIN_NAME` and `EMAIL_ACCOUNT`
values:

```bash
docker-compose run --rm --entrypoint "\
  certbot certonly \
    -d DOMAIN_NAME -d *.DOMAIN_NAME \
            --agree-tos \
            --non-interactive \
            --email EMAIL_ACCOUNT \
            --dns-route53 \
            --preferred-challenges=dns \
            --rsa-key-size 4096 \
            --server https://acme-v02.api.letsencrypt.org/directory" certbot
```

## Start the Akaunting Application

Run: `docker-compose up -d`

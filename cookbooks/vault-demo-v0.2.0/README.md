# chef-vault-demo-cookbook

## Some of the prerequisites (for demonstrators/contributors only)

- I've created an s3 bucket, `s3://chef-vault-demo` that has two objects,
  - unikitten.png (1.7Mb)
  - minikitten.png (417Kb)
- I've created an AWS user, `chef-vault-demo` which has the following policy applied, so it can do nothing except GET those objects
```
Show Policy
 {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1454343707000",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::chef-vault-demo/unikitten.png",
                "arn:aws:s3:::chef-vault-demo/minikitten.png"
            ]
        }
    ]
}
```
- this cookbook runs a lot faster using `inspec` and `kitchen-dokken`. The command `rake dokken` symlinks `_kitchen.local.yml` to `.kitchen.local.yml` so you can use. As of 1 Feb 2016, this required local installation of the gem from https://github.com/chef/kitchen-inspec so you can use Inspec against docker.

## 0: Doing it all in cleartext

See video to see the details

- git clone this repo
- update the inspec tests
- update the recipe
- test-kitchen

## 1: Use a template and variables

1. Move content to `templates/default/s3cfg.erb`
2. Use variables `aws_access_key` and `aws_secret_key`
3. Set variables and use them in them template

## 2: Use a data bag

```
git checkout v2-initial-databag
git diff origin/v1 -- recipes
git diff origin/v1 -- data_bags
```

In the recipe we've replaced the variable assignments with a fetch from a data bag:

```
aws = data_bag_item('cleartext', 'aws')
aws_secret_key = aws['aws_secret_key']
aws_access_key = aws['aws_access_key']
```

and we've now created a databag, as JSON, at data_bags/cleartext/aws.json, with contents:

```
{
  "id": "aws",
  "aws_access_key": "AKIAJWLDGDWB6HVRMRAQ",
  "aws_secret_key": "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"
}
```


## 3: Encrypted data bags

```
git checkout v3-encrypted-databag
```

Make the secret from random data

```
mkdir -p files/default/
openssl rand -base64 512 |
  tr -d '\r\n' > files/default/encrypted_data_bag_secret
```

For testing, we'll use the `-z`option to `knife` for local-mode operations (a.k.a 'chef-zero'). To create an encrypted data bag from our cleartext `aws.json`:

```
knife data bag -z create encrypted
knife data bag -z from file encrypted data_bags/cleartext/aws.json --secret-file files/default/encrypted_data_bag_secret
```

and we'll update our recipe....

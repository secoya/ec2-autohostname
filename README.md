ec2-autohostname
=====================

This startup script utilizes the _hostname_ tag of an ec2 instance
to set its hostname and update the corresponding entry in Route 53.

# Installation #
## Preparation ##
You will need to modify the script a wee bit.
* Set the `$domain` variable to your desired domain/subdomain.
* Create a user and a group for the purpose of updating the DNS. A restrictive group policy for that purpose can be found in `ec2-authostname-policy`.
* Adjust the hosted zone ID in the policy to match the ID of your Route 53 hosted zone domain/subdomain. The ID is found in the main overview of hosted zones.
* Adjust the AWS credentials to match the newly created user. Namely `$AWS_ACCESS_KEY_ID` and `$AWS_SECRET_ACCESS_KEY`.

## Manual setup ##

```
cp ec2-autohostname /etc/init.d/ec2-autohostname
chmod 755 /etc/init.d/ec2-autohostname
insserv -d ec2-autohostname
```

### Dependencies ###
The script depends on some python libraries namely:
* boto
* cli53
Just run the following to install them.

```
apt-get install python-pip
pip boto
curl -sL -o /root/cli53.tar.gz https://github.com/barnybug/cli53/tarball/0.3.0
tar -xvf cli53.tar.gz
easy_install barnybug-cli53-99dd79e
```

The easy_install part is neccessary because there seems to be a packaging problem with cli53.

## Bootstrapping ##
An installation script when bootstrapping is included, the only neccessary environment variable needed is `$imagedir`.
$imagedir should point to the root of the bootstrapped installation.
You can use it with [ec2debian-build-ami](https://github.com/andsens/ec2debian-build-ami) by specifying the `--script` parameter like so:
```
./ec2debian-build-ami --script ec2-autohostname/install-ec2-autohostname
```

# Usage #
Simply specify a tag named _hostname_ in your AWS EC2 interface containing the desired subdomain as a value.
When the instance is started the hostname should automatically be updated.
You can also update a running instance by simply running /etc/init.d/ec2-autohostname manually.
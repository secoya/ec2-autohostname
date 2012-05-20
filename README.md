ec2-autohostname
=====================

This startup script utilizes the `hostname` tag of an ec2 instance
to set its hostname and update the corresponding entry in Route 53.

# Installation #
## Preparation ##
* Create a user and a group for the purpose of updating the DNS. A restrictive group policy for that purpose can be found in `ec2-authostname-policy`.
* Adjust the hosted zone ID in the policy to match the ID of your Route 53 hosted zone domain/subdomain. The ID is found in the main overview of hosted zones.

The settings.template holds three parameters that can (and must!) be customized.
* Adjust the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` to match the newly created user.
* Adjust the `ZONE` variable to match the zone ID you retrieved in the preparation step.

## Using the install script ##
The install script works for debian squeeze only!

Run `./install-ec2-authostname` and you are done.

## Manual setup ##
If you are not using the install script, you will need to modify `ec2-authostname` by hand.
The parameters are located at the top of the script.
After that:

1. Copy the script to `/etc/init.d/`
2. chmod it, so that the script and -- most importantly -- the AWS credentials can only be read by root
3. Use `insserv` to make it run at boot

```
cp ec2-autohostname /etc/init.d/ec2-autohostname
chmod 700 /etc/init.d/ec2-autohostname
insserv -d ec2-autohostname
```

### Dependencies ###
The script depends the `boto` python library.
The library in the debian package repository does not support Route 53, so you will need to install it via pip:
```
apt-get install -y python-pip
pip install boto
```

## Bootstrapping ##
An installation script when bootstrapping is included, the only neccessary environment variable needed is `$imagedir`.
`$imagedir` should point to the root of the bootstrapped installation.
You can use it with [ec2debian-build-ami](https://github.com/andsens/ec2debian-build-ami) by specifying the `--script` parameter like so:
```
./ec2debian-build-ami --script ec2-autohostname/install-ec2-autohostname
```

# Usage #
Specify a tag named `hostname` in your AWS EC2 interface containing the desired subdomain as a value.
When the instance is started the hostname should automatically be updated.
Upon shutdown the CNAME is automatically removed.
If the hostname is already taken the script will fail and not try to rebind it.
Run `/etc/init.d/ec2-authostname status` to see what the configured hostname points to and which records point to the current instance.

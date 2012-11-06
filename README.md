ec2-autohostname
=====================

This startup script utilizes the `hostname` tag of an ec2 instance
to set its hostname and update the corresponding entry in Route 53.

# Installation #
## Preparation ##
1. Create a user and a group for the purpose of updating the DNS. A restrictive group policy for that purpose can be found in `ec2-authostname.policy`.
2. Adjust the hosted zone ID in the policy to match the ID of your Route 53 hosted zone domain/subdomain. The ID is found in the main overview of hosted zones.
3. ``ec2-autohostname.template`` holds three parameters that can (and must!) be customized. Don't modify the file directly, copy it to ``ec2-autohostname`` and edit that file.
4. Adjust the `aws_access_key_id` and `aws_secret_access_key` to match the newly created user.
5. Adjust the `zone_id` variable to match the zone ID you retrieved in the second preparation step.

*All the variables in the script are marked with Xs.*

## Using the install script ##
The install script works for debian squeeze only!

Run `./install-ec2-autohostname` and you are done.

## Manual setup ##

1. Copy the script to `/etc/init.d/`
2. chmod it, so that the script and -- most importantly -- the AWS credentials can only be read by root
3. Use `insserv` to make it run at boot

```
cp ec2-autohostname /etc/init.d/ec2-autohostname
chmod 700 /etc/init.d/ec2-autohostname
insserv -d ec2-autohostname
```

### Dependencies ###
The script depends on the `boto` python library.
The library in the debian package repository does not support Route 53, so you will need to install it via pip:
```
apt-get install -y python-pip
pip install boto
```

## Bootstrapping ##
An installation plugin for [ec2debian-build-ami](https://github.com/andsens/ec2debian-build-ami) is included.
Include it by specifying the `--plugin` parameter like so:
```
./ec2debian-build-ami --plugin ec2-autohostname/bootstrap-ec2-autohostname
```

# Usage #
Specify a tag named `hostname` in your AWS EC2 interface containing the desired subdomain as a value.
When the instance is started the hostname should automatically be updated.
Upon shutdown the CNAME is automatically removed.
If the hostname is already taken the script will fail and not try to rebind it.
Run `/etc/init.d/ec2-autohostname status` to see what the configured hostname points to and which records point to the current instance.

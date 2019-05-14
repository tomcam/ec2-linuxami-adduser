# Adding a user to an Amazon Linux 2 EC2 Instance

Scenario: You hired a programmer. You want to give 
him access to your EC2 Instance running Amazon Linux 2 
so that he can create and run programs in Go.

In concept you're going to give that person a username and
password, but EC2 doesn't like passwords. Instead you'll assign
a username and provide a public key file (calld a PEM file)
to use instead of  a password. 
The new user will log in to ssh by passing it his local copy of the 
PEM file, which must match an entry in a file called `authorized_keys`, 
stored in a hidden directory namd `~/.ssh` you create with the
new user account.

## Log into EC2

* Open the Terminal program on your Mac.

* Assuming my the DNS name of your instance is `ec2-user@ec2-50-51-232-66.us-east-1.compute.amazonaws.com` 
and your PEM key is named ec2.pem, and located in the directory `~/.ssh`, then you'd log in this way from
your Mac:

```bash
# Remember:
# Replace ~/.ssh/ec2.pem with the name of the PEM
# file you generated for your EC2 instance.
# Replace ec2-user with your username, and 
# Replace ec2-50-51-232-66.us-east-1.compute.amazonaws.com with the
# public DNS for your instance.
ssh -i "~/.ssh/ec2.pem" ec2-user@ec2-50-51-232-66.us-east-1.compute.amazonaws.com
```

## Create the new user

Suppose the new user is named coolio.

```bash
# Create the new user named coolio.
sudo adduser coolio

# Give coolio sudo privileges.
sudo su - coolio
```

The prompt changes, because you are now acting 
as the user named Coolio.

## Create a directory for the public key

When the new user logs in via ssh, access to the public key
is required.

```bash
# Create a directory for the public key.
mkdir .ssh

# Give new user full access to this directory, but
# prevent the outside world from touching it.
chmod 700 .ssh

# Create a file named authorized_keys.
touch .ssh/authorized_keys

# Make this file private.
# Allow only the creator of this file 
# to edit it.
chmod 600 .ssh/authorized_keys
```

## Get the public key for your instance

* Leave the EC2 instance for Coolio open and open a new Terminal instance on your Mac.

The new user needs a copy of the public key to your instance
on his private machine. It will match an entry in `authorized_keys`
in his `~/ssh` directory on the EC2 instance.

To retrieve the public key you can use the command 
`ssh-keygen -y -f <filename>`, so in the example here it would be 
`ssh-keygen -y -f ~/.ssh/ec2.pem`. To copy its contents to the clipboard,
you pipe to the `pbcopy`. 

* Run the full command:

```bash
# Replace ~/.ssh/ec2.pem with the name of the PEM
# file you generated for your EC2 instance.
ssh-keygen -y -f ~/.ssh/ec2.pem | pbcopy
```

Now



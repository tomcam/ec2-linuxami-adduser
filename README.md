# Adding a user to an Amazon Linux 2 EC2 Instance

Scenario: You hired a programmer. You want to give 
him access to your EC2 Instance running Amazon Linux 2 
so that he can create and run programs in Go.

In concept you're going to give that person a username and
password, but EC2 doesn't like passwords. Instead you'll:

* Create a username and home directory
* Generate a private key file (calld a PEM file)
to use instead of  a password. 
* Generate from that
private key file a separate public key file. 
* Add the public key file contents to a a file called in the new user's 
account in the EC2 instance `authorized_keys`, 
stored in a hidden directory namd `~/.ssh` you create with the
new user account. 
* Send the private key to the new user.

The `authorized_keys` file itself is
tricky to create, because its permissions must be set to much
more restrictive levels than normal and its contents must be
carefully formatted.

This process will require you to open up 2 terminals.
One will be on your local machine, and the other will be on the EC2 instance. 
You will log in as yourself on the EC2 instance, then morph into
the new user, create `~/.ssh/authorized_keys`, and add the public
key to it.

## Log into EC2 and create a private key file

* Log into the EC2 dashboard

* Under **NETWORK & SECURITY** on the navigation panel, choose **Key Pairs**.

* Choose **Create Key Pair**

* Give it a name, say, `coolio`.

### Important! Save the private key file

You're given the chance to save the private key file. You absolutely need to save it.

* Save the private key file to your local machine. 
By convention it is saved in a directory named ~/.ssh, so
you might save the file as `~/.ssh/coolio.pem.`

### Set the private key file permissions to 600

Amazon doesn't want your new user's private key file to be available to anyone else.

* Set its permission to 600:

```bash
# Make this file private.
# Allow only the creator of this file 
# to edit it.
chmod 600 ~/.ssh/coolio.pem
```

* Obtain its public key and copy to the system clipboard:

```bash
# -y means read a private key, obtain its public key,
#    and display the contents to the standard output device.
# -f is the file from which to obtain the private key.
#  | pbcopy send standard output to the Macintosh clipboard.
#    You can omit the  | pbcopy to see the public key.
ssh-keygen -y -f ~/.ssh/coolio.pem | pbcopy
```




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

Now the file has been copied to your system clipboard. It's time to
add an entry to the `authorized_keys` file on the EC2 instance.

## Create the authorized_keys entry

* Return to the



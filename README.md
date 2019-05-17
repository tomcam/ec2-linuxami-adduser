# Adding a user to an Amazon Linux 2 EC2 Instance

Scenario: You hired a programmer. You want to give 
them ssh access to your EC2 Instance running Amazon Linux 2 
so that they can create and run programs in Go or
PHP or whatever. This tutorial shows how to add a user
and give them ssh access.

## Who's this tutorial for?

In my opinion everything written about ssh is either too high level, omitting important details,
or written by people who don't really understand it, and who therefore leave out
the "why" of what they're telling you to do. I tried to write a
howto for the Amazon Linux 2 sysadmin who needs to bring someone
onboard but who isnt an ssh expert. If I don't explain every step
clearly, please email me at tomcampbell@gmail.com and I'll do my 
best to make it better.

## It's about PEM files, not passwords

In concept you're going to give that person a username and
password, but EC2 doesn't like passwords. It requires
specially generated files be passed into the `ssh` command
line. These files contain a programmatically generated "private key".
Instead of creating a password you will:

* Create a username and home directory
* Use the EC2 dashboard to generate a private key file (calld a PEM file)
to use instead of  a password. You will create a special 
private key exclusively for the new user. Doing so will 
allow you to revoke the user's privileges simply by removing
their private key.
* Generate from that private key file a separate public key file. 
* Add the public key file contents to a a file called in the new user's 
directory  `~/.ssh/authorized_keys`. 
* Send the private key file to the new user, who should store it in 
their own `~/.ssh` directory.

The user will then log in using the copy of the .PEM you 
sent instead of a password.

The `authorized_keys` file itself is
tricky to create, because its permissions must be set to much
more restrictive levels than normal and its contents must be
carefully formatted. Same with the user's PEM file
in their `.ssh` directory.

This process will require you to open up 2 terminals.
One will be on your local machine, and the other will be on the EC2 instance. 
You will log in as yourself on the EC2 instance, then morph into
the new user to finish your tasks.

## Log into your EC2 instance

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

# Give the user a password.
# I suggest you note it in another file and paste it in.
# You are asked for the password twice.
sudo passwd coolio

# Give coolio sudo privileges.
# Run this program:
sudo visudo
# And add the following new line.
coolio ALL=(ALL)NOPASSWD:ALL

# Now become the new user, coolio
sudo su - coolio
```

The prompt changes, because you are now acting 
as the user named Coolio, on the EC2 instance.

## Removing a user account/deleting a user

You can delete the user just as easily as you created them.

* Log into your EC2 instance


```bash
# Don't do this unless you want to
# remove the user!
# In this example, the username is coolio.
# Default command preserves coolio's home directory.
sudo userdel coolio

# To erase Coolio's home directory, 
# add the -r option:
sudo userdel -r coolio
```



## Create a directory for the public key

When the new user logs in via ssh, access to the public key
is required. Amazon requires that you restrict
as much access to that key as possible.

* Create a directory called ~/.ssh for the public key,
and a file within it named authorized_keys

```bash
# Create a directory for the public key.
mkdir ~/.ssh

# Give new user full access to this directory, but
# prevent the outside world from touching it.
chmod 700 ~/.ssh

# Create a file named authorized_keys.
touch .ssh/authorized_keys

# Make this file private.
# Allow only the creator of this file 
# to edit it.
chmod 600 .ssh/authorized_keys
```

## Log into EC2 and create a private key file

* Log into the EC2 dashboard

* Under **NETWORK & SECURITY** on the navigation panel, choose **Key Pairs**.

* Choose **Create Key Pair**

* Give it a name, say, `coolio`. 
The file doesn't need to be the same as the username. 
It's just convenient for tracking.

### Important! Save the private key file

You're given the chance to save the private key file. You absolutely need to save it.

* Save the private key file to your local machine. 

By convention it is saved in a directory named `~/.ssh`, so
you might save the file as `~/.ssh/coolio.pem.`

### Set the private key file permissions to 600

Amazon doesn't want your new user's private key file to be available to anyone else.

* Set the .PEM file's permission to 600:

```bash
# Make this file private.
# Allow only the creator of this file 
# to edit it.
chmod 600 ~/.ssh/coolio.pem
```

## Create the public key and copy it to the system clipboard

The public key is derived from the private key. You create it
by running a utility program named `ssh-keygen`.

* The following command will obtain public key 
from the PEM file and copy its contents to the system clipboard:

```bash
# -y means read a private key, obtain its public key,
#    and display the contents to the standard output device.
# -f is the file from which to obtain the private key.
#  | pbcopy send standard output to the Macintosh clipboard.
#    You can omit the  | pbcopy to see the public key.
ssh-keygen -y -f ~/.ssh/coolio.pem | pbcopy
```

## Add the public key to authorized_keys on the EC2 instance

The public key is now on the system clipboard. Let's get it into
the new user's `authorized_keys` file on the EC2 instance.

* Return to the EC2 instance, where you are logged in as the 
new user, and append the contents of the clipboard to 
the `.ssh/authorized_keys` file, something like 
what you see below.

### Warning

If you're not sure what's happening, read this section carefully a couple of times.

Right now the contents of your clipboard contain the public key in the form of
two strings separated by a newline. 
They look something like this. 
(You can just paste it to the command line to see.)

```
ssh-rsa AAAAB3NzaC1yc2EAAAAAZRR2mWaPfUlSJwljop+5cicVxP1QEz2XiilWLrn2DTexkfQDE+Q1YmgKAd9ImGEyV+YCRC0eULJ4ZJzdL/g0NQEV4R3Tj4d5vQQGn9je4Yy91xEZDB0xd7DxYjr5p58iZZVjPXCSDkbeySzS+/THSB0W+6PLJosHdzVeXfbKcHpDrLY8fjncVIUpcCjSZ/ddKaTZ/T76C+ncPrjjmK9VmeYYP/JHvp5o2HQHVeBi3RvObcON/zrlQUwEocRe96CTjS+7aG4qk9wsc/Ofaaba2I67P0SRYbnlgFRpTRe6T2/1kBPwvgAL7WNU4XiDsSlhrf6X91juowthwCcrJR21
```

* Fire up a text editor and paste these contents to the file `~/.ssh/authorized_keys`. 

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCH2mWaPfUlSJwljop+5cicVxP1QEz2XiilWLrn2DTexkfQDE+Q1YmgKAd9ImGEyV+YCRC0eULJ4ZJzdL/g0NQEV4R3Tj4d5vQQGn9je4Yy91xEZDB0xd7DxYjr5p58iZZVjPXCSDkbeySzS+/THSB0W+6PLJosHdzVeXfbKcHpDrLY8fjncVIUpcCjSZ/ddKaTZ/T76C+ncPrjjmK9VmeYYP/JHvp5o2HQHVeBi3RvObcON/zrlQUwEocRe96CTjS+7aG4qk9wsc/Ofaaba2I67P0SRYbnlgFRpTRe6T2/1kBPwvgAL7WNU4XiDsSlhrf6X91juowthwCcrvQ7NFK1
```

### Optional: add an identifier after the public key

Suppose you want a reminder that this is Coolio's public key. You can append a space and an identifier after the
text of your public key, like this:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCH2mWaPfUlSJwljop+5cicVxP1QEz2XiilWLrn2DTexkfQDE+Q1YmgKAd9ImGEyV+YCRC0eULJ4ZJzdL/g0NQEV4R3Tj4d5vQQGn9je4Yy91xEZDB0xd7DxYjr5p58iZZVjPXCSDkbeySzS+/THSB0W+6PLJosHdzVeXfbKcHpDrLY8fjncVIUpcCjSZ/ddKaTZ/T76C+ncPrjjmK9VmeYYP/JHvp5o2HQHVeBi3RvObcON/zrlQUwEocRe96CTjS+7aG4qk9wsc/Ofaaba2I67P0SRYbnlgFRpTRe6T2/1kBPwvgAL7WNU4XiDsSlhrf6X91juowthwCcrvQ7NFK1 coolio
```

### Important note if you're coming back to do this again to an existing authorized_keys

This is written as if you're in the middle of a tutorial where you know you've just created a
user, which means there's no existing `.ssh/authorized_keys` file.
If you're repeating this process at a later date and need to leave the previous public key
in this file, be sure that you add the next public key  directly below, with no empty
lines:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCH2mWaPfUlSJwljop+5cicVxP1QEz2XiilWLrn2DTexkfQDE+Q1YmgKAd9ImGEyV+YCRC0eULJ4ZJzdL/g0NQEV4R3Tj4d5vQQGn9je4Yy91xEZDB0xd7DxYjr5p58iZZVjPXCSDkbeySzS+/THSB0W+6PLJosHdzVeXfbKcHpDrLY8fjncVIUpcCjSZ/ddKaTZ/T76C+ncPrjjmK9VmeYYP/JHvp5o2HQHVeBi3RvObcON/zrlQUwEocRe96CTjS+7aG4qk9wsc/Ofaaba2I67P0SRYbnlgFRpTRe6T2/1kBPwvgAL7WNU4XiDsSlhrf6X91juowthwCcrvQ7NFK1 coolio
ssh-rsa MMMADFasdfadfBAAQDFASDFLAKDFJASDKFDASDFP1QEz2XiilWLrn2DTexkfQDE+Q1YmgKAd9ImGEyV+YCRC0eULJ4ZJzdL/g0NQEV4R3Tj4d5vQQGn9je4Yy91xEZDB0xd7DxYjr5p58iZZVjPXCSDkbeySzS+/THSB0W+6PLJosHdzVeXfbKcHpDrLY8fjncVIUpcCjSZ/ddKaTZ/T76C+ncPrjjmK9VmeYYP/JHvp5o2HQHVeBi3RvObcON/zrlQUwEocRe96CTjS+7aG4qk9wsc/Ofaaba2I67P0SRYbnlgFRpTRe6T2/1kBPwvgAL7WNU4XiDsSlhrf6X91juowthwCcrvQ7NFK1 tom
```

You don't want it to look like this:

### Don't do this! Blank lines between public keys may cause trouble

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCH2mWaPfUlSJwljop+5cicVxP1QEz2XiilWLrn2DTexkfQDE+Q1YmgKAd9ImGEyV+YCRC0eULJ4ZJzdL/g0NQEV4R3Tj4d5vQQGn9je4Yy91xEZDB0xd7DxYjr5p58iZZVjPXCSDkbeySzS+/THSB0W+6PLJosHdzVeXfbKcHpDrLY8fjncVIUpcCjSZ/ddKaTZ/T76C+ncPrjjmK9VmeYYP/JHvp5o2HQHVeBi3RvObcON/zrlQUwEocRe96CTjS+7aG4qk9wsc/Ofaaba2I67P0SRYbnlgFRpTRe6T2/1kBPwvgAL7WNU4XiDsSlhrf6X91juowthwCcrvQ7NFK1 coolio


ssh-rsa MMMADFasdfadfBAAQDFASDFLAKDFJASDKFDASDFP1QEz2XiilWLrn2DTexkfQDE+Q1YmgKAd9ImGEyV+YCRC0eULJ4ZJzdL/g0NQEV4R3Tj4d5vQQGn9je4Yy91xEZDB0xd7DxYjr5p58iZZVjPXCSDkbeySzS+/THSB0W+6PLJosHdzVeXfbKcHpDrLY8fjncVIUpcCjSZ/ddKaTZ/T76C+ncPrjjmK9VmeYYP/JHvp5o2HQHVeBi3RvObcON/zrlQUwEocRe96CTjS+7aG4qk9wsc/Ofaaba2I67P0SRYbnlgFRpTRe6T2/1kBPwvgAL7WNU4XiDsSlhrf6X91juowthwCcrvQ7NFK1 tom
```

## Get a copy of the .pem file to the new user

The new user gets a copy of the private key .PEM file, then adds
it to the right directory and sets file permissions to 
be more secure. Here's how:

* Email or deliver in some more secure way the .PEM file (`coolio.pem`, in this case) to the
user you've created. Explain that it should be stored in `~/.ssh` on their machine, so for
example it would be the file `~/.ssh/coolio.pem`.

### Set permissions

* Tell them to set the permissions for the new file to 600:

```bash
# Make this file private.
# Allow only the creator of this file 
# to edit it.
chmod 600 ~/.ssh/coolio.pem
```

* Explain that they will ssh in something like this:

```bash
# Remember:
# Replace ~/.ssh/coolio.pem with the name of the 
# new user's PEM file.
# Replace coolio with the new username, and 
# Replace ec2-50-51-232-66.us-east-1.compute.amazonaws.com with the
# public DNS for your instance.
ssh -i "~/.ssh/coolio.pem" coolio@ec2-50-51-232-66.us-east-1.compute.amazonaws.com
```

The first time they log in they'll get a warning because they are logging into
an address they haven't encountered before:

```
The authenticity of host 'coolio@ec2-50-51-232-66.us-east-1.compute.amazonaws.com (178.99.20.213)' can't be established.
ECDSA key fingerprint is SHA256:v5wGzOxtO/nI9rsXAfdkadfjtqntkQBTbrY9mjHQ.
Are you sure you want to continue connecting (yes/no)?
```

* The user should enter `yes` to avoid this warning in the future.

They're given a final notice that this could be suspect:

````
Warning: Permanently added ec2-50-51-232-66.us-east-1.compute.amazonaws.com
````

## What if WARNING: UNPROTECTED PRIVATE KEY FILE appears instead?

It is possible that this message will appear instead:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/Users/coolio/.ssh/coolio.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/Users/coolio/.ssh/coolio.pem": bad permissions
coolio@c2-50-51-232-66.us-east-1.compute.amazonaws.com: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

If you see it, it means you didn't [set permissions](#set-permissions). 

* Just return to the [set permissions](#set-permissions) and repeat your
steps from there, starting with the `chmod 600 ~/.ssh/coolio.pem` part.

## TODO:

* Explain visudo actions
* Create a group, then Add user to that group with slightly reduced permissions so they can't see other home dirs, etc. https://serverfault.com/questions/283558/creating-user-accounts-in-amazon-ec2 https://superuser.com/questions/739643/amazon-linux-add-user-to-group  https://www.sweetprocess.com/procedures/eG30mkvYDrfAmevj78A0i6E1GZE/add-an-administrator-to-your-amazon-aws-account/

## Reference

AWS: [Managing User Accounts on Your Linux Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/managing-users.html#add-user-best-practice)
Hackernoon: [Add New Users to EC2 and Give SSH Key Access](https://hackernoon.com/add-new-users-to-ec2-and-give-ssh-key-access-d2abd084f30c)
Garron.me: [Take Control of your Linux sudoers file](https://www.garron.me/en/linux/visudo-command-sudoers-file-sudo-default-editor.html)

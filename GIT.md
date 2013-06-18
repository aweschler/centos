## Overview

This page explains one way to set up a Git repository on a CentOS server, followed by one way to set up Gitosis on that server. 
This page assumes that you have set up the server using the instructions in BASE.md.

I developed these instructions to help me create multiple separate remote git repositories for students of my courses to submit assignments.

## Git/Gitosis Installation

Run the following as root to install required packages.

    yum install git-core python-setuptools

Do the following as non-root.

    cd ~/src
    git clone https://github.com/tv42/gitosis.git
    cd gitosis
    sudo python setup.py install
	
Now a user needs to be created that will own the repositories, commonly named git.  An example command to create this user is as follows:

     sudo adduser --system --shell /bin/sh --group --home /home/git git

A password is not required, but can be used.  The most important parameter in this command is making sure the user has a valid shell as defined by --shell.

## SSH Key Generation and Authentication

From the admin's computer, an ssh authentication key must be created. On the local machine, this is done via the following command:

     ssh-keygen -t rsa

The above command will create a .pub file that is the public key, found in the directory $HOME/.ssh/id_rsa.pub.  This .pub file then needs to be copied to the server, ideally into the /tmp folder for permission reasons.  From there, that key needs to be used in initializing Gitosis.  The user with this ssh key will have admin control over the Git repositories and users.  The initialization command is:

     sudo -H -u git gitosis-init < /tmp/id_rsa.pub

Now, from the local machine where the admin generated their ssh key from, the following commands can be used to download necessary files for control of Gitosis:

     git clone git@serverhostname:gitosisadmin.git
     cd gitosis-admin

This will download two files, a gitosis.conf file and a /keydir directory.  Modification of the .conf file will control repositories themselves along with user privileges and grouping, and the /keydir directory will be used to hold the .pub files from user created when they generate their public ssh keys.

## Test Scenario

As a test scenario, there are three users:  Alice, Bob, and Zed.  Alice and Bob are students and Zed is the isntructor.  There are two repositories, one which only Alice and Zed have access to, and one which only Bob and Zed have access to.

__SSH Key Copying__

All three users will need to have installed Git, and followed the instructions to generate their public ssh key.  Those .pub files needs to be copied into the /keydir directory by the admin (which can be done by simply copying them into the downloaded /keydir directory on his local machine and committing later).  The name of the file minus the .pub will become their username within Gitosis.  These files can be copied into the /keydir and added into the repository with the following commands:

     cd gitosis-admin
     cp ~/alice.pub keydir/
     cp ~/bob.pub keydir/
     cp ~/zed.pub keydir/
     git add keydir/alice.pub keydir/bob.pub keydir/zed.pub

	 
__Creating the Repositories__

From there the two repositories need to be created, named assignmentA and assignmentB.  From the admin's machine, this can be done with the following commands:

     mkdir assignmentA
     cd assignmentA
     git init
     git remote add origin git@serverhostname:assignmentA.git

From there do some work in the repository with the standard add and commits.  Finally, the command

     git push origin master:refs/heads/master

Will push all changes to the assignmentA Git repository to the server.

The process can be repeated for the assignmentB repository (and getting out of any repositories):

     mkdir assignmentB
     cd assignmentB
     git init
     git remote add origin git@serverhostname:assignmentB.git
     (work)
     git push origin master:refs/heads/master

	 
__Modifying User Privileges__
	 
Now that the two repositories have been created, two groups need to be created in the gitosis.conf file.  Open the gitosis.conf file in a text editor, and add the following block of text:

     [group teamA]
     members = alice zed
     writable = assignmentA

     [group team B]
     members bob zed
     writable = assignmentB

From there those changes can be committed with the following commands:

     git commit -a -m "Allow alice and zed write access to assignmentA, allow bob and zed write access to assignmentB"
     git push

The desired repositories can now be cloned onto Zed's, Alice's, or Bob's computer using the following command:

     git clone git@serverhostname:assignmentA.git

Or

     git clone git@serverhostname:assignmentB.git

The ability to read and write will be controlled by the privileges allowed through Gitosis, so Alice will have neither read or write access to assignmentB and Bob will have neither read or write access to assignmentA.

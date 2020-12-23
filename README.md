# Push-To-Repo-From-Gitlab-Runner
Sample `.gitlab-ci.yml` to push changes to the repository on which the pipeline is running

##
 Create a new SSH key pair using `ssh-keygen`
 
 Do not add a passphrase to the SSH key, or the job will prompt for it
 
 Your keys will be generated and stored in ~/.ssh by default
 
 The file which ends in a .pub extension is the public key and the corresponding file contains the private key
 
 Create a new CI Variable in Settings > CI/CD > Variables
 
 As Key enter the name SSH_PRIVATE_KEY and in the Value field paste the content of your private key that you created earlier
 
 Find out the host keys of your gitlab server by running the `ssh-keyscan <HOSTNAME>` command from a terminal on a trusted network
 
 Create another CI Variable with Key as SSH_KNOWN_HOSTS and Value as the output of `ssh-keyscan <HOSTNAME>` command

##

 Run scripts which will modify existing files or generate new files
 Pass the modified or generated files as artifacts so that consequent jobs can use them

##

 Hardcode dependency to the previous job and fetch modified or generated files to be committed to branch

 Install ssh-agent if not already installed, required by Docker

 Run ssh-agent inside the build environment

 Add the SSH key stored in SSH_PRIVATE_KEY variable to the agent store

 Create the SSH directory and give it the right permissions

 Add the host keys of your gitlab server to known hosts and give it the right permisssions

 You can optionally disable host key checking, be aware that by disabling host key checking you are susceptible to man-in-the-middle attacks
 
 WARNING: Use this only with the Docker executor

 Since we are using Git, configure email, name, and origin of your repository

 Checkout the current branch where the pipeline is being run

 Could also use git add . or git add * depending on the use case

 It is important to add [ci skip] to commit message otherwise there 
 is a threat of infinite pipelines being triggered

 For versions of Git >= 2.10 we can also use ci.skip option

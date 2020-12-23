# Push-To-Repo-From-Gitlab-Runner
Sample `.gitlab-ci.yml` to push changes to the repository on which the pipeline is running


```
## Create a new SSH key pair using `ssh-keygen`
## Do not add a passphrase to the SSH key, or the job will prompt for it
## Your keys will be generated and stored in ~/.ssh by default
## The file which ends in a .pub extension is the public key and the corresponding file contains the private key
## Create a new CI Variable in Settings > CI/CD > Variables
## As Key enter the name SSH_PRIVATE_KEY and in the Value field paste the content of your private key that you created earlier
## Find out the host keys of your gitlab server by running the `ssh-keyscan <HOSTNAME>` command from a terminal on a trusted network
## Create another CI Variable with Key as SSH_KNOWN_HOSTS and Value as the output of `ssh-keyscan <HOSTNAME>` command


## Run scripts which will modify existing files or generate new files
generate-or-modify-files:
  stage: generate

  script:
      - bash generate-files-for-source-control-update.sh
      - bash modify-existing-files.sh

  ## Pass the modified or generated files as artifacts so that consequent jobs can use them
  artifacts:
    paths:
      - /paths/to/generated/files
      - /paths/to/modified/files

update-branch:
  stage: update

  when: manual
  
  ## Hardcode dependency to the previous job and fetch modified or generated files to be committed to branch
  needs:
    - generate-or-modify-files

  before_script:
    ## Install ssh-agent if not already installed, required by Docker
    - command -v ssh-agent >/dev/null || ( apt-get update -y && apt-get install openssh-client -y )
    
    ## Run ssh-agent inside the build environment
    - eval $(ssh-agent -s)
    
    ## Add the SSH key stored in SSH_PRIVATE_KEY variable to the agent store
    ## We're using tr to fix line endings which makes ed25519 keys work without extra base64 encoding
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    
    ## Create the SSH directory and give it the right permissions
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    
    ## Add the host keys of your gitlab server to known hosts and give it the right permisssions
    - echo "$SSH_KNOWN_HOSTS" >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    
    ## You can optionally disable host key checking 
    ## Be aware that by disabling host key checking you are susceptible to man-in-the-middle attacks
    ## WARNING: Use this only with the Docker executor
    ## if you use it with shell you WILL overwrite your user's SSH config
    # - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" >> ~/.ssh/config
    
    ## Since we are using Git, configure email, name, and origin of your repository
    - git config --global user.email "<runner's email>"
    - git config --global user.name "<runner's name>"
    - git remote set-url --push origin $(perl -pe 's#.*@(.+?(\:\d+)?)/#git@\1:#' <<< $CI_REPOSITORY_URL)

  script:
    - git status
    
    ## Checkout the current branch where the pipeline is being run
    - git checkout "${CI_COMMIT_REF_NAME}"
   
    ## Could also use git add . or git add * depending on the use case
    - git add /paths/to/generated/files /paths/to/modified/files
   
    ## It is important to add [ci skip] to commit message otherwise there 
    ## is a threat of infinite pipelines being triggered
    - git commit -m "[ci skip] ${CI_COMMIT_MESSAGE}" || true
    
    ## For versions of Git >= 2.10 we can also use ci.skip option
    - git push origin "${CI_COMMIT_REF_NAME}" -o ci.skip || true
```

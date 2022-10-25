# Set Up Continuous Integration/Deployment from Development to Production Servers
This guide will help you set up CI/CD in an environment with Ubuntu development and production servers. Completion of all steps should allow for the following process:

- Developer makes some changes on development site.
- Developer commits and pushes their changes to master branch of a GitHub repository.
- GitHub automatically connects to the production server and pulls the changes.

![CI/CD Process](/images/ci-cd-graphic.png?raw=true "CI/CD Process")

# Setup

## Production Server

### Create a Production SSH Key
Generate a key to be used by a) the GitHub actions runner to connect to the production server and run a git pull command and b) the production server to authenticate access to the private repository on GitHub containing the production code base. You can use two separate keys if you want, but for ease of use I will use the same key for both purposes.

1. Log into the production server as the user that you want to make the git pull and run `ssh-keygen`. ([official SSH docs](https://www.ssh.com/academy/ssh/keygen) as needed). Do not add a password unless you intend to add a password parameter to the `.yml` script (see [this link](https://github.com/appleboy/ssh-action) if so).
2. Copy down the values of your shiny new key files. Retrieve via `cat ~/.ssh/id_rsa` (private key) and `cat ~/.ssh/id_rsa.pub` (public key). 

### Add Production Server's Private Key to Authorized Keys

1. Add the content of `~/.ssh/id_rsa.pub` to `~/.ssh/authorized_keys`. This will allow the GitHub runner we're about to set up to connect to the server.

### Verify Git is Present and Using SSH

1. Navigate to the directory that the application files live in. 
2. Verify that a `.git` directory is present. If there isn't use `git fetch` to grab new files (like the needed `.git` directory) from the remote repository.
3. Use `git config --list` to verify that the remote repo is a SSH remote like `git@github.com:{you}/{repo}.git`.
    * If it **IS** you're good.
    * If it **IS NOT** (i.e. it is an http remote like `https://github.com/{owner}/{repo}.git`) use `git remote set-url {remote_name} git@github.com:{owner}/{repo}.git` to change it to whatever SSH remote is listed for the repository on [GitHub](https://github.com). This will allow the production server to pull from GitHub using its SSH key for authentication (we will add the public key to GitHub in a future step).
4. Add the fingerprint of GitHub's server to the production server by connecting to some SSH remote. **If and only if you know the production repo is already up to date** perform a `git pull` to test the SSH remote and add GitHub's fingerprint to the server's known_hosts. Otherwise, consider doing a `git clone` of a random **SSH** repository elsewhere. You can immediately delete the files, but this will add GitHub fingerprint to the server's known_hosts file. 

### Get PWD of Project
1. Run `pwd` (print working directory) inside the directory that contains the `.git` directory. Copy the result down as you will need it in a later step.

## Development Server

### Create Workflow Script

1. Log on to the development server.
2. Navigate to the development directory (the one with the `.git` directory).
3. Create a `.github` folder. Inside it, create a `workflows` folder.
4. Inside `workflows` create a `.yml` script. This will define the workflow. The content of the script should be a copy of or very similar to that in this repository (located at `.github/workflows/update-production.yml`).
5. **DON'T YET PUSH TO GIT!** You'll need to modify the GitHub repository and GitHub account in the next steps before this script can successfully execute.

## GitHub

### Add SSH Key to GitHub Account
In order for the production server to connect to a private GitHub repository and pull any changes the production server will need to authenticate itself. We do this via the SSH key we set up on the production server earlier. 

1. Log into GitHub as the user you want to perform the pull. This user will need to be able to access the repository you wish to pull from.
2. Navigate to https://github.com/settings/keys.
3. Select the "New SSH key" button.
4. Add an informative title (such as "user@productionserver ci/cd auth key") and the public key of the production server (the one we found using `cat ~/.ssh/id_rsa.pub`). This will allow the production server to authenticate itself as this account and perform a pull.

### Add Secrets to GitHub Repository
Note that the development script utilizes GitHub secrets, for example `${{ secrets.PRODUCTION_SERVER }}`. These secrets must be added to the GitHub repository. To do so follow these steps:

1. Browse to the repository.
2. Browse to `https://github.com/{owner}/{repo}/settings/secrets/actions` (or click the appropriate menu items to get there).
3. Select "New repository secret" and add each of the following (assuming you have not modified the `update-production.yml` script):
    - name: `PRODUCTION_SERVER`, value: The domain name of the production server.
    - name: `PRODUCTION_USER`, value: The user that should be used to ssh into the production server.
    - name: `SSH_PRIVATE_KEY`, value: The entire contents of the production server's private ssh key generated previously, i.e. the contents we retrieved with `cat ~/.ssh/id_rsa`.
    - name: `PRODUCTION_PWD`, value: The result of running `pwd` in the top level directory of the production environment.

## Watch the Magic Happen
### (Or, more likely, begin debugging.)
At this point if you've completed all the steps correctly you should be able to go back to your development site and make a `git push`. Give the runner about 30 seconds to do its work and then check your production environment. Provided that everything worked `.github/workflows/actions-script-name-here.yml` should appear in your production environment. (If not, navigate to the Actions panel of the repository and examine the action to see what might have gone wrong... Happy debugging!) If it does appear, then you're set and any further pushes to your master branch on GitHub will automagically be deployed to the development environment.
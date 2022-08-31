# Continuous Integration/Deployment from Development to Production
This repository takes advantage of GitHub actions. Following is a rundown of the procedure for setting up CI/CD for an environment with a development and production server.

## Set Up CI/CD

### Generate Public/Private SSH Key Pair
Unless you want to use an existing public/private key, you'll need to generate one. Refer to the [official SSH docs](https://www.ssh.com/academy/ssh/keygen). Do not add a passphrase unless you intend to modify the parameters of `update-production.yml` (if you do intend to modify the script see [GitHub Action SSH](https://github.com/marketplace/actions/ssh-remote-commands)). Hang onto the public and private key pair to use shortly.

### Add Public SSH Key to Production Server
1. Log onto the production server as the user you'll want to pull changes to the live environment.
2. Add the public SSH key you generated to the user's "approved_keys" file at `~/.ssh/approved_keys`. This will allow the GitHub runner we're about to set up to connect to the production server.

### Set Up Workflow Script
1. Log on to the development server.
2. Navigate to the development directory (the one with the `.git` directory).
3. Create a `.github` folder. Inside it, create a `workflows` folder.
4. Inside `workflows` create a `.yml` script. This will define the workflow. The content of the script should be similar to that in this repository located at `.github/workflows/update-production.yml`.
5. **DON'T YET PUSH TO GIT!** You'll need to modify the GitHub repository in the next steps beore this script can successfully execute.

### Add Secrets to GitHub Repository
Notice that the script utilizes GitHub secrets, e.g. `${{ secrets.PRODUCTION_SERVER }}`. These secrets must be added to the GitHub repository. To do so follow these steps:
1. Browse to the repository.
2. Browse to `https://github.com/your-repo/settings/secrets/actions` (or click the appropriate menu items to get there).
3. Select "New repository secret" and add each of the following (assuming you have not modified the `update-production.yml` script):
    - name: `PRODUCTION_SERVER`, value: The domain name of the production server.
    - name: `PRODUCTION_USER`, value: The user that should be used to ssh into the production server.
    - name: `SSH_PRIVATE_KEY`, value: The entire contents of the private ssh key generated previously.

## GitHub Actions Definitions
In case you get confused. I know I did. :)
- `Workflow`: An automated process that will run one or more `jobs` as specified on a particular `event`. Worflows live at `.github/workflows` in a repository. 
- `Event`: A specific activity that may happen in a repository. Can be set up to trigger a `workflow` run.
- `Jobs`: Steps in a `workflow` executed by the same `runner`.
- `Actions`: Custom GitHub Actions application that performs a complex task that will generally be repeated a number of times.
- `Runners`: A server that runs `workflows`.
# bitbucket-to-github-mirror

Synchronize (and mirror) a Bitbucket repository to GitHub.

This uses Pipelines the Bitbucket integrated CI/CD service to automate remote mirroring after every push.

### What is Required?

-   Access to configure repositories on both Bitbucket and GitHub.
-   A OpenSSH generated `ssh-ed25519` keypair (not PuTTY).
-   Mirror URL to GitHub repository (same as clone with ssh).
-   Available Bitbucket Pipelines minutes (50 minutes free per month).

### How does it work?

Clones the repository into a newly created directory, creates remote-tracking branches for each branch in the cloned repository and creates and checks out an initial branch that is forked from the cloned repository’s currently active branch.

Fetches all remotes, then removes any remote-tracking references that no longer exist. Tags are fetched due to an explicit refspec because the remote was cloned using the mirror option, and are also subject to pruning.

Mirror specifies that all refs under refs/ (which includes but not limited to refs/heads/, refs/remotes/, and refs/tags/) be mirrored to the remote repository. Newly created refs will be pushed to the remote end, updated refs will be force updated on the remote end, and deleted refs will be removed from the remote end.

## Before getting started

> Never share or provide the private key publicly!

Create the required OpenSSH keypair with the following command using Linux, Windows 10, or Windows 11. If prompted to enter passphrase, leave empty, just hit `[Enter]` or `[Return]` on the keyboard for no passphrase.

```shell
ssh-keygen -t ed25519 -C "Bitbucket to GitHub Mirror" -f git-mirror-ed25529 -N ''
```

The newly generated public and private keys are saved to your current working directory.

## Setup GitHub

1. Create a new repository for the mirror.
2. Add the generated public key under `Settings` > `Security` > `Deploy keys` > `Add deploy key`, then select the checkbox to `Allow write access`.

## Setup and configure Bitbucket

1. Enable Pipelines under `Repository settings` > `Pipelines` > `Settings`.
2. Add the generated keypair (public and private keys) under `Repository settings` > `Pipelines` > `SSH keys`.
3. From the same page, add `github.com` as the known host address, then click `Fetch` followed by `Add host`.
4. Add the public key under `Repository settings` > `Security` > `Access keys` > `Add key`.
5. Inside the Bitbucket repo, create a bitbucket-pipelines.yml file containing the following:

> Replace `<GITHUB_GIT_SSH_REMOTE>` with the GitHub repository URL being used for the mirror.

```yml
pipelines:
    default:
        - step:
              name: "Synchronize and mirror this Bitbucket repository to Github..."
              image: atlassian/default-image:latest
              clone:
                  enabled: false
              script:
                  - git clone --mirror $BITBUCKET_GIT_SSH_ORIGIN $BITBUCKET_CLONE_DIR
                  - cd $BITBUCKET_CLONE_DIR
                  # - git lfs fetch --all
                  - git fetch --all --prune
                  - git remote set-url --push origin <GITHUB_GIT_SSH_REMOTE>
                  - git push --mirror
                  # - git lfs push --all
```

### Does Large File Support work?

Git Large File Support is provided by both Bitbucket and Github but must be enabled on the repositories before you uncomment the commands in your pipeline configuration. The pipeline will fail with runtime errors if included and `git lfs` is not enabled on both.

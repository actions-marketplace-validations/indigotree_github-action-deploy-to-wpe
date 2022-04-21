# GitHub Action to deploy WordPress projects to WP Engine

This GitHub Action is used to deploy code from a GitHub repo to a WP Engine environment of your choosing. Deploy a full site directory or a sub-directory of your WordPress install. Other options include performing a PHP Lint, custom rsync flags, clearing cache, and a executing a post-deploy script of your choosing.

## Setup Instructions

Follow along with the [video tutorial here!](https://wpengine-2.wistia.com/medias/crj1lp3qke)

1. **MAIN.YML SETUP**

- Copy the following `main.yml` to `.github/workflows/main.yml` in your root of your local WordPress project/repo, replacing values of `PRD_BRANCH`, `PRD_ENV` for the branch and WPE Environment name of your choice. Optional vars can be specified as well. Consult ["Environment Variable & Secrets"](#environment-variables--secrets) for more available options.

2. **SSH PRIVATE KEY SETUP IN GITHUB**

- [Generate a new SSH key pair](https://wpengine.com/support/ssh-keys-for-shell-access/#Generate_New_SSH_Key) if you have not already done so.

- Add the _SSH Private Key_ to your [Repository Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) or your [Organization Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-an-organization). Save the new secret "Name" as `WPE_SSHG_KEY_PRIVATE`.

**NOTE:** If using a GitHub Organization, adding the SSH key to the Organization Secrets will allow all repos to reference the same SSH key for deploys using the method in the sample `main.yml`. The SSH Key also connects to all installs made available to its WP Engine User. One key can then effectively be used to deploy all projects to their respective sites on WP Engine. Less work. More deploys!

3. **SSH PUBLIC KEY SETUP IN WP ENGINE**

- Add _SSH Public Key_ to WP Engine SSH Gateway Key settings. [This Guide will show you how.](https://wpengine.com/support/ssh-gateway/#Add_SSH_Key)

  **NOTE:** This Action DOES NOT utilize WP Engine GitPush or the GitPush SSH keys [found here.](https://wpengine.com/support/git/#Add_SSH_Key_to_User_Portal)

4. Git push your site GitHub repo. The action will do the rest!

View your actions progress and logs by navigating to the "Actions" tab in your repo.

## Example GitHub Action workflow

### Simple main.yml:

```
name: Deploy to WP Engine
on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: GitHub Action Deploy to WP Engine
      uses: wpengine/github-action-wpe-site-deploy@v2.3.5
      with:

      # Deploy vars
        WPE_SSHG_KEY_PRIVATE: ${{ secrets.WPE_SSHG_KEY_PRIVATE }}

      # Branches & Environments
        PRD_BRANCH: main
        PRD_ENV: prodsitehere
```

### Extended main.yml

```
name: Deploy to WP Engine
on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: GitHub Action Deploy to WP Engine
      uses: wpengine/github-action-wpe-site-deploy@v2.3.5
      with:

      # Deploy vars
        WPE_SSHG_KEY_PRIVATE: ${{ secrets.WPE_SSHG_KEY_PRIVATE }}
        PHP_LINT: TRUE
        FLAGS: -azvr --inplace --delete --exclude=".*" --exclude-from=.deployignore
        CACHE_CLEAR: TRUE
        TPO_SRC_PATH: "wp-content/themes/genesis-child-theme/"
        TPO_PATH: "wp-content/themes/genesis-child-theme/"

      # Branches & Environments
        PRD_BRANCH: main
        PRD_ENV: prodsitehere

        STG_BRANCH: feature/stage
        STG_ENV: stagesitehere

        DEV_BRANCH: feature/dev
        DEV_ENV: devsitehere
```

## Environment Variables & Secrets

### Required

| Name                   | Type    | Usage                                                                              |
| ---------------------- | ------- | ---------------------------------------------------------------------------------- |
| `PRD_BRANCH`           | string  | Insert the name of the GitHub branch you would like to deploy from, example; main. |
| `PRD_ENV`              | string  | Insert the name of the WP Engine environment you want to deploy to.                |
| `WPE_SSHG_KEY_PRIVATE` | secrets | Private SSH Key for the SSH Gateway and deployment. See below for SSH key usage.   |

### Optional

| Name           | Type   | Usage                                                                                                                                                                                                                                               |
| -------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `STG_BRANCH`   | string | Insert the name of a staging GitHub branch you would like to deploy from. Note: exclude leading / from branch names.                                                                                                                                |
| `STG_ENV`      | string | Insert the name of the WP Engine Stage environment you want to deploy to.                                                                                                                                                                           |
| `DEV_BRANCH`   | string | Insert the name of a development GitHub branch you would like to deploy from. Note: exclude leading / in branch names.                                                                                                                              |
| `DEV_ENV`      | string | Insert the name of the WP Engine Dev environment you want to deploy to.                                                                                                                                                                             |
| `PHP_LINT`     | bool   | Set to TRUE to execute a php lint on your branch pre-deployment. Default is `FALSE`.                                                                                                                                                                |
| `FLAGS`        | string | Set optional rsync flags such as `--delete` or `--exclude-from`. The example is excluding paths specified in a `.deployignore` file in the root of the repo. This action defaults to a non-destructive deploy using the flags in the example above. |
| `CACHE_CLEAR`  | bool   | Optionally clear cache post deploy. This takes a few seconds. Default is TRUE.                                                                                                                                                                      |
| `TPO_SRC_PATH` | string | Optional path to specify a theme, plugin, or other directory source to deploy from. Ex. `"wp-content/themes/genesis-child-theme/"` . Defaults to "." Dir.                                                                                           |
| `TPO_PATH`     | string | Optional path to specify a theme, plugin, or other directory destination to deploy to. Ex. `"wp-content/themes/genesis-child-theme/"` . Defaults to WordPress root directory.                                                                       |

### Further reading

- [Defining environment variables in GitHub Actions](https://docs.github.com/en/actions/reference/environment-variables)
- [Storing secrets in GitHub repositories](https://docs.github.com/en/actions/reference/encrypted-secrets)
- As this script does not restrict files or directories that can be deployed, it is recommended to leverage one of [WP Engine's .gitignore templates.](https://wpengine.com/support/git/#Add_gitignore)

### Legal

See the LICENSE file for license information.

Copyright (C) 2021-present, Indigo Tree Digital Ltd.

This project was based on prior work from https://github.com/wpengine/github-action-wpe-site-deploy
Copyright (c) 2021 WP Engine


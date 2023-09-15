# Microservices-Best-Practices

## Github

To prepare the Github repository to have the needed configuration in order to work properly, you will need to enable some permissions and security options.

### Permissions

In order to push packages or perform other actions using the automatic `GITHUB_TOKEN` generated during the workflow, you will need following steps to add needed permissions:

* Add **Github Token** permission. Go to `Settings > Actions > General` and enabling Workflow permissions as `Read and write permissions.`
* Add current repository to allow Package Management.
  * Go `Packages` tab in the main Github page
  * Enter into the desired Package with the `Permission error` and select `Package settings` option.
  * On the `Manage Actions access` click `Add repository` and select the repository with the Github action that will publish the artifact. Wait until the repository is added and change its role to `Write`.

### Security

For each reposirtory it can be enabled some tools that will be used during Git events (push, pull_request, etc..) and Github actions.

* To Enable `Dependabot` in the repository. Go to `Settings > Code Security & analysis > Dependabot alerts` and `enable` the feature.
Also configure Secret Scanning with the repository. 
* To enable `Secrets Scanning` Go to `Settings > Code Security & analysis` and `enable` it . Enable `Push Protection` as well which will prevent secrets from being pushed to the repository.

### Github Pages

Enable Github pages in the repository.
Go to `Settings -> Pages` and select the `gh-pages` branch that will be used to publish the static content and used by a Github action with the Documentation generated. Finally click `save` so a new Workflow that will be triggered to publish the website.

```txt
Your site is live at https://jsa4000.github.io/Docs-as-Code-Example/
```

> NOTE: Wait until the URL is generated If the branch does not appear, create a new branch as `gh-pages`.

### Github Actions

Github actions are enabled by default having workflows inside a folder `.github/workflows`. In these workflows you have to set the trigger actions (`on: [events...]`) and the jobs and steps that are part of this. Another alternative is using the Github Actions Templates provided by Github `Actions -> New Workflow`. After selecting a Workflow this will create a new workflow file that can be committed directly within the ui.

In order to test these workflows locally you can use `act` tool that enabled the capability to test workflows using docker simulating the same runner's environment.

### Act

When you run [act](https://github.com/nektos/act) it reads in your **GitHub Actions** from `.github/workflows/` and determines the set of actions that need to be run. It uses the `Docker API` to either pull or build the necessary images, as defined in your workflow files and finally determines the execution path based on the dependencies that were defined.

GitHub Actions offers managed virtual environments for running workflows. In order for act to run your workflows locally, it must run a container for the runner defined in your workflow file. Act allow to use different [images](https://github.com/nektos/act#runners) for the runners.

#### Installation

In order to install act you can use the main packages managers such as brew, choco, nix, etc..

```bash
# Install on MacOS
brew install act

# Install a specific version
brew install act --HEAD

# Test act installation
act --version  

# It can be also installed as a Github CLI extension
gh extension install https://github.com/nektos/gh-act
```

Initialize act and pull the base images for the runners.

```bash
# Run act (select image size large, medium or micro depending on the compatibility)
act

# Run with verbose option to watch the progress.
act -v

# This will trigger the 'on: push' event in the branch, so matched Github Actions will be triggered.

# Using M1 chips you will need to use '--container-architecture linux/amd64'
act -l --container-architecture linux/amd64

```

> When running act for the first time, it will ask you to choose image to be used as default. It will save that information to `~/.actrc`.

`~/.actrc`

```bash
-P ubuntu-latest=catthehacker/ubuntu:full-latest
-P ubuntu-latest=catthehacker/ubuntu:full-20.04
-P ubuntu-18.04=catthehacker/ubuntu:full-18.04
```

If the act process sucks at the start, try to pull the images manually using the container engine.

```bash
# Manually Pull the image
 docker pull catthehacker/ubuntu:full-latest
```

#### Basic Commands

The basic commands are:

```bash
# Command structure:
act [<event>] [options]
If no event name passed, will default to "on: push"
If actions handles only one event it will be used as default instead of "on: push"

# act help
act ---help

# List all actions for all events:
act -l

# List the actions for a specific event:
act workflow_dispatch -l

# List the actions for a specific job:
act -j test -l

# Run the default (`push`) event:
act

# Run a specific event:
act pull_request

# Run a specific job:
act -j test

# Collect artifacts to the /tmp/artifacts folder:
act --artifact-server-path /tmp/artifacts

# Run a job in a specific workflow (useful if you have duplicate job names)
act -j lint -W .github/workflows/checks.yml

# Run in dry-run mode:
act -n

# Enable verbose-logging (can be used with any of the above commands)
act -v

# To use a different image for the runner, use the -P option.
act -P ubuntu-18.04=nektos/act-environments-ubuntu:18.04
act -P ubuntu-18.04=nektos/act-environments-ubuntu:18.04 -P ubuntu-latest=ubuntu:latest -P ubuntu-16.04=node:16-buster-slim
```

#### Configuration

`act` supports loading environment variables from an `.env` file. The default is to look in the working directory for the file but can be overridden by:

```bash
# Run act by providing an environment file
act --env-file my.env
```

`.env`

```bash
MY_ENV_VAR=MY_ENV_VAR_VALUE
MY_2ND_ENV_VAR="my 2nd env var value"
```

To run act with secrets, you can enter them interactively, supply them as environment variables or load them from a file. The following options are available for providing secrets:

```bash
# Use somevalue as the value for `MY_SECRET``.
act -s MY_SECRET=somevalue

# Check for an environment variable named `MY_SECRET` and use it if it exists. 
# If the environment variable is not defined, prompt the user for a value.
act -s MY_SECRET

# load secrets values from my.secrets file.
# secrets file format is the same as .env format
act --secret-file .secrets
```

`.secrets`

```bash
GITHUB_TOKEN=MY_SECRET_TOKEN
```

If your workflow depends on this `GITHUB_TOKEN,` you need to create a personal access token and pass it to act as a secret:

```bash
# Set the GITHUB_TOKEN by parameters
act -s GITHUB_TOKEN=mysecrettoken
```

#### Examples

> The current folder must be on the root path where the `.github/workflows` is placed.

```bash
# Get all the Jobs
act -l

# Run a demo Job
act -j Explore-GitHub-Actions --container-architecture linux/amd64
```

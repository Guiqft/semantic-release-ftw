# Automatic Semantic Releases

This repository demonstrates how to set up automatic releases for a Github repository using [semantic-release](https://github.com/semantic-release/semantic-release). It's designed around a workflow where each commit on the `homolog` branch creates a new release candidate (rc), and each commit on `main` triggers a new latest release.

## Workflows

To achieve the desired behavior, two Github workflows are needed:

### 1. Create Release PR

This workflow is responsible for discovery the next release version and create a pull request for it. It gets triggered manually by a `workflow_dispatch` event and can only be triggered on the `dev` or `release/*` branches. It can create two types of PRs:

#### Prerelease PR

It happens when the workflow is triggered from `dev`, creating a pull request from `release/vX.x.x` to `homolog`, where all the prereleases happens. 

#### Release PR

When the workflow is triggered from a release branch (matching `release/*`), it creates a pull request from `release/vX.x.x` to `main`, where all the latest releases should be. 

> 💡 Tip: Is there a option to skip the release branch flow in emergencial situations, where `dev` can be streamlined into `main`, creating a fast release. 

#### Workflow Steps

The workflow needs to follow a few steps in order to work in the desired way: 

- Determines the branch name based on which branch the workflow was triggered.
- Checks out the code.
- Rebase the release branch into the current branch, keeping a linear git history, necessary to discovery the next release version.
- Install dependencies.
- Determines the next version with `semantic-release --dry-run` command.
- Checks if a new release is needed.
- Creates a release branch named `release/vX.x.x` (ignoring the suffix `-rc` in case of a prerelease).
- Creates a pull request with the changes for the new release.
- Closes previous opened release pull requests (to prevent major/minor/patch releases coexisting).

### 2. Release

This workflow is responsible for creating the release. It gets triggered whenever a commit is made to the `main` or `homolog` branches (normally the commits coming from the release pull request previouly created). It's responsible to automatically run `semantic-release`, creating a release commit that updates `package.json` and `CHANGELOG.md`.

The workflow steps:

- Checks out the code.
- Installs dependencies.
- Runs semantic-release to create the release.
- Updates the `homolog` branch if a new release was published from `main` (this prevents `rc` releases being created for a already published release).
- Deletes the release branch `release/vX.x.x` when the release is published on main.

## Contributing

Contributions to improve these workflows are welcome. Please create a pull request with your proposed changes.

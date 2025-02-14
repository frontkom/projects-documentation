# Deployment for environments with Development / Production environments.

> A note about the main branch. The main branch should be `main` in most newer projects. Some old projects will use the branch `master`. In this document the name `main` will be used from here on out to describe the main branch, so keep that in mind if your project uses the branch `master`.

> A note about naming. Some projects will refer to this model as having `staging` and `production`. In this document the name `development` will be used from here on out to describe the "other" environment. 

## What git revisions are pointing to which environments.

### Development

The `develop` branch will be automatically deployed to Development. This should be the default branch, and you should merge new features and bug fixes into it.

### Production

The production environment should have the latest tag deployed. This tag will have the pattern `release-YYYY-MM-DD-<num>`. This tag should correspond to a commit on the `main` branch.

## How to deploy to different environments

### Development

Merge a new PR into the `develop` branch.

### Production

Merge a new PR into the `main` branch. This will in turn automatically trigger a new release to be made, and this release should automatically be deployed to production.

You can also open a PR automatically either on a schedule, or by running the included workflow. The workflow should be named `open-prod-pr.yml`.

To start a release process, do as follows:

From the repo frontpage, click "Click here to create a release to production" from the README. This should take you directly to the workflow. In the top right you should see a button saying "Run workflow".

![Screenshot from 2025-01-08 12-32-53](https://github.com/user-attachments/assets/d1170732-5fc7-44d4-8262-125033df14d9)

Click the button "Run workflow" and accept the default values.

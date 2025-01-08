# Deployment for environments with Development / Staging / Production

> A note about the main branch. The main branch should be `main` in most newer projects. Some old projects will use the branch `master`. In this document the name `main` will be used from here on out to describe the main branch, so keep that in mind if your project uses the branch `master`.

## What git revisions are pointing to which environments.

### Development

The `develop` branch will be automatically deployed to Development. This should be the default branch, and you should merge new features and bug fixes into it.

### Staging

The `main` branch will be automatically deployed to Staging. To deploy to staging, you open a PR from `develop` towards `main` and merge in the changes. The changes should be merged with the `merge`.

### Production

The production environment should have the latest tag deployed. This tag will have the pattern `release-YYYY-MM-DD-<num>`. This tag will usually correspond to a commit on the `main` branch.

## How to deploy to different environments

### Development

Merge a new PR into the `develop` branch.

### Staging

Merge a new PR into the `main` branch. This will usually be a sync of `develop` into `main`.

### Production

Use the included workflow to automatically tag and release the `main` branch. This should be a workflow called "Create a release and tag on main".

To use the workflow do as follows:

From the repo frontpage, click "Create release" from the README. This should take you directly to the workflow. In the top right you should see a button saying "Run workflow".

![Screenshot from 2025-01-08 12-32-53](https://github.com/user-attachments/assets/d1170732-5fc7-44d4-8262-125033df14d9)

Click the button "Run workflow" and accept the default values.

The release should now be created in the expected way, and should be listed with the releases of the repository

![Screenshot from 2025-01-08 12-33-58](https://github.com/user-attachments/assets/9dfccc3d-868b-4b49-ac06-8a25fe7f51f0)

In the Deployments list, you should now (or soon) see a deployment happening on Production

![Screenshot from 2025-01-08 12-35-08](https://github.com/user-attachments/assets/b7edd528-cefd-45b7-8335-c8c74433d1bf)

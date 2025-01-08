# Deployment for environments with dev / staging / production

> A note about the main branch. The main branch should be `main` in most newer projects. Some old projects will use the branch `master`. In this document the name `main` will be used from here on out to describe the main branch, so keep that in mind if your project uses the branch `master`.

## What git revisions are pointing to which environments.

### Dev

The `develop` branch will be automatically deployed to dev. This should be the default branch, and you should merge new features and bug fixes into it.

### Staging

The `main` branch will be automatically deployed to staging. To deploy to staging, you open a PR from `develop` towards `main` and merge in the changes. The changes should be merged with the `merge`.

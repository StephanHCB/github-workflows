# github-workflows

This is a collection of GitHub workflows.

## docker-build-push

Workflow to build from a Dockerfile and push the resulting container to the GHCR.

Use as follows (put a `.yml` file in `.github/workflows/` with these contents).

```
name: Create and publish Docker image

on:
  push:
    branches:
      - 'main'

jobs:
  call_build-and-push-image:
    permissions:
      contents: read
      packages: write
    uses: StephanHCB/github-workflows/.github/workflows/docker-build-push.yml@v0.1.0
    with:
      image-name: ${{ github.repository }}
      image-tags: latest
      full-repo-url: https://github.com/${{ github.repository }}
      branch-or-tag-name: ${{ github.ref_name }}
      commit-hash: ${{ github.sha }}
      registry-user: ${{ github.actor }}
    secrets:
      registry-pass: ${{ secrets.GITHUB_TOKEN }}
```

**Important:** Please ensure:
- you refer to a release version or, even better, a commit hash in this repository, NOT the mainline. In the example
  above, I reference tag `v0.1.0`. You should NEVER run any GitHub workflow from a branch that can just change to
  something an attacker may have pushed.
- you limit the permissions of the standard GITHUB_TOKEN as shown above to just repo contents read and packages write.

_Why write a custom workflow?_

The standard workflow mentioned in the documentation installs node.js to basically run the
exact same shell commands as this workflow
- adds large amounts of unnecessary attack surface (on something that by design has write access to your packages)
- is needlessly obscure and hard to debug when it fails

### Inputs / Secrets

| Input/Secret               | Required? | Default | Explanation                                                                                                                                                                                                             |
|----------------------------|-----------|---------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| registry                   | no        | ghcr.io | Docker registry to push the finished image to.                                                                                                                                                                          |
| image-name                 | yes       |         | The image name to push. Can be set to `${{ github.repository }}`, which will evaluate to e.g. StephanHCB/repo-name                                                                                                      |
| image-tags                 | no        | latest  | Space separated list of tag(s) to push, for example `v1 v1.3 v1.3.7 latest`.                                                                                                                                            |
| full-repo-url              | yes       |         | The full repository URL. Should usually be set to `https://github.com/${{ github.repository }}`. The image will be labeled with this information, and if pushing to ghcr.io, the description page will be auto-filled.  |
| branch-or-tag-name         | yes       |         | The branch or tag name to clone, in short format. Should usually be set to `${{ github.ref_name }}`.                                                                                                                    |
| commit-hash                | yes       |         | The commit hash. Should usually be set to `${{ github.sha }}`. The image will be labeled with this information.                                                                                                         |
| registry-user              | yes       |         | Username to use for docker registry login. If ghrc.io, you probably want `${{ github.actor }}`.                                                                                                                         |
| registry-pass **(Secret)** | yes       |         | Token to use for docker registry login. If ghcr.io, should usually be set to `${{ secrets.GITHUB_TOKEN }}`, and do remember to limit the token permissions to package write and content read and nothing else. |

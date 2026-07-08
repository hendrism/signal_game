# GitHub Pages Deployment Design

## Goal

Publish the current `signal.html` from `main` to GitHub Pages automatically after every push without maintaining generated site files in the source branch.

## Architecture

A GitHub Actions workflow triggered by pushes to `main` will:

1. Check out the repository.
2. Extract the JavaScript from `signal.html` and run `node --check`.
3. Copy `signal.html` to `_site/index.html` and add `_site/.nojekyll`.
4. Configure GitHub Pages and upload `_site` as the Pages artifact.
5. Deploy the artifact with GitHub's official Pages deployment action.

The workflow will use `contents: read`, `pages: write`, and `id-token: write` permissions. A deployment concurrency group will prevent overlapping Pages deployments while allowing the latest queued deployment to finish.

## Branch Strategy

`main` remains the only development branch. GitHub's Pages artifact deployment mechanism replaces the generated `gh-pages` branch workflow. The stale remote `gh-pages` branch will be deleted only after the first successful deployment is verified.

## Failure Handling

Syntax-check, artifact preparation, upload, or deployment failures will stop the workflow and remain visible in GitHub Actions. A failed run will not replace the last successful Pages deployment.

## Verification

Before committing, validate the workflow YAML structure, run the same JavaScript syntax check locally, and verify the prepared artifact contains `index.html` and `.nojekyll`. After pushing, verify the Actions deployment succeeds and the public Pages URL serves the current `main` content, including the corrected zero fragment costs for the 2-hour and 8-hour expedition technologies.

## Scope

This change adds only the Pages workflow and this design record. It does not alter game behavior, dependencies, build tooling, or save data.

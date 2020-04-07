# CD4PE Workflow Migration

## Summary

This document outlines best practices for converting existing Git workflows to being CD4PE compatible. Since standard CD4PE workflows require that users have a `master` branch which contains a linear series of commits, some multi-branch workflows can be difficult reconcile to this way of working due to the fact that a single branch (`master`) needs to contain all commits in the repo in order for them to be deployable.

This document does not recommend what specific workflow should be used once CD4PE is in place, just how to get CD4PE implemented without causing changes in your environment.

## Expectations

This best practice expects that the customer is using some type of long-lived branch workflow wherein there are a number of branches which code proceeds through sequentially before reaching `production` (or similar). Workflows that rely on functionality that cannot be achieved in [r10k](https://github.com/puppetlabs/r10k) are not likely to be supported by this process. Some examples of workflows that this approach would be compatible with would be:

### Long-Lived Branches

Most long-lived branch workflows consist of a few branches which nodes are assigned to long-term. An example of these branches could be:

* `production`
* `staging`
* `development`

Usually changes are worked on in "feature branches" which contain a single feature and are reviewed though a pull request into one of the long lived branches. The destination of these pull requests depends on when the code integration is supposed to happen, however the exact details of the workflow does not matter.

### Trunk Workflows

Some workflows use a central branch such as `development` which features branches are taken from and merged back into. Occasionally releases are taken from this branch and merged into another long-lived branch such as `production`. Sometimes this also is combined with a "release branch" system where the release branch is tested before being merged. It is also possible to combine this with many other long-lived branches that the release would pass through before making it to production.

## Best Practice Details

Let's assume that we are beginning from a state where we have three branches with the following refs:

| Branch | SHA | Logical Order |
|--------|-----|---------------|
| production | `34211f1` | 3rd |
| staging | `f0f25e0` | 2nd |
| development | `593a1e3` | 1st |

In order to ensure that we can have CD4PE deploy new bookkeeping branches with the same revisions as the existing branches (and therefore not cause any unnecessary changes) we need to create a `master` branch which contains all of the commits in the repository.

### Perform the Merge-Back

Before creating the `master` branch, perform a merge-back. To do this we need to merge the long lived branches back onto each other in reverse order. In the above environment this would mean:

* Create a pull request from `production` to `staging`
* Review the changes and merge the PR
* Create a pull request from `staging` to `development`

In each PR the only changes that should appear in the diff are ones that were not present in the environment above but **not** in the environment below. As long is the logical order was followed, this should never happen. There will however likely be a large number of merge commits. If there are changes, these should be reviewed and merged.

Once all branches have been merged back to the 1st branch (`development`), take a note of the new refs:

| Branch | SHA | Logical Order |
|--------|-----|---------------|
| production | `34211f1` | 3rd |
| staging | `5380862` | 2nd |
| development | `52c74cb` | 1st |

### Create the `master` branch

Once the merge-back is complete, create a branch named `master` from the 1st logical branch (`development`). This should contain all of the commits that the current long-lived branches are using. This can be checked using the following command which will return the name of the master branch if it contains the commit.

```shell
$ git branch --contains e0aaea8
  development
* master
```

### Create the bookkeeping branches

Now we are ready to create the bookkeeping branches for CD4PE. Create new branches from the old ones, this essentially duplicates the existing branches. Remember to deploy these branches to the Puppet master.

```shell
git checkout -b cd4pe_production production
git checkout -b cd4pe_staging staging
git checkout -b cd4pe_development development
```

### Creating Node Groups

Now that the bookkeeping branches have been created, we either need to create new node groups or use existing ones. It should be possible to move nodes from their previous branches to the new ones since there should be no changes, this can be confirmed using:

```shell
git diff cd4pe_production production
```

Once all nodes are running against `cd4pe_` branches we can create a CD4PE pipeline and we are done! Since the merge-back the master branch is a linear history of commits, further changes should be merged into this branch from feature branches. The CD4PE pipeline will then move the head of the bookkeeping branches forward along the master branch, as it is designed to.

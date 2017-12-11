# Collaborating with upstream

## Updating the module upstream

To update the module [upstream](https://github.com/voxpupuli/puppet-misp) you have to fork the repository, clone it in your machine, then apply the changes needed. To finish the process you need to push to your fork and then generate a merge request to the upstream module.

## Updating module on your organisational Git repository

The steps to update the module in your organisational Git repositories from the upstream GitHub module are:

- Clone the Git repository from your organisational Git server in your computer.
- Set both remotes (origin - institutional Git repository && GitHub - your for repo)
- To updated form GitHub you will need to do a subtree pull:
  + `git subtree pull -P code github features` For this we assume that `github` is the remote for your fork in `github` and features is the branch in which you are working or want to update from. You might need to do a merge, there is no support for rebase in subtree pull (this is unfortunate).
- Push / Push force to your organisational Git repository and run Puppet in your computer.

# git Team Workflow

When we work in teams, the way we use git becomes very important. There are two main approaches to working in a team:

1. Each team member **forks** the main repo. When they want to add some code to the code base, they make a pull request (much like when you submit your homework), which needs to be reviewed by the lead developer, before being merged into the code.

1. Each team member **clones** the repo. This means that everyone is effectively working on the same repo. In this case if a team member wants to add some code, they simply push their commit.

The first method is safer. There is far less chance of team members overwriting other's code. However it requires a lead developer who can spend their day reviewing code and managing the main codebase. For this reason it is slightly slower, and more expensive, since it requires a more senior developer who does not really do any coding on the project.

The second method is much quicker, since all developers can code, however it is more dangerous, because every developer is pushing to the same repo, so code can easily be lost.

The first method is usually favoured by start-ups and agencies where there is an established hierarchy in place. The second method is usually favoured by open-source projects, where every developer is considered at the same level.

For your group project, we'll be using the second method, where everyone pushes to the same repo.

To start with you will need to nominate a _git master_. This person will create the original repo that all team members will use. They will also be responsible for deploying the final app.

## Branching

Up until this point in the course you have been using one branch: _master_ to work with. However when multiple developers are working with the same repo, pushing to _master_ can be problematic, since developers can inadvertently overwrite each other's code.

To mitigate this we can create different _branches_ to work from. A branch is a parallel set of commits. By using branches developers can work in parallel confident that they will not interfere with anyone else's code.

This is fine, however, at some point we need to bring all the code together. This is called _merging_, and this has the potential for _conflicts_ where code on two different branches would overwrite each other.

To ensure that we manage merging safely as possible, at GA we have a specific way of working with branches. Check out the [Team Git Cheatsheet](team-git-cheatsheet.pdf) that will take you through the workflow.

## Creating the repo

1. Create a repo on GitHub with the following settings:

  ![Example GihHub repo](https://media.git.generalassemb.ly/user/15120/files/f252e880-d86f-11e9-8cfe-6bad7474f752)

  Make sure you have checked _Initialize this repository with a README_.

1. Go to _Settings > Collaborators_ and add each member of your team

  ![Adding collaborators](https://media.git.generalassemb.ly/user/15120/files/9dfc3880-d870-11e9-9b20-e8ea0cddf3ab)

1. Clone the repo to your laptop

1. Create a new branch called `development`, then push it to GitHub:

  ```
  gco -b development
  git push origin development
  ```

## Using the repo

1. Once each team member has accepted the invite to the repo (they should have been sent an email invite) they can then clone the repo to their laptops

1. Once cloned, fetch the development branch like so:

  ```
  git fetch
  gco development
  ```

  Now all team members should be working on the same repo, with a _master_ and _development_ branch.

1. When a developer wants to add a feature, they should create a feature branch. It's important that they create the feature branch _from development_:

  ```
  gco development
  gco -b <feature-branch>
  ```

  > **Note**: Replace `<feature-branch>` with the name of the feature you are working on eg: `login-form`

1. Once the feature is completed, merge `development` into the feature, _before_ merging the feature into development.


For a more detailed workflow, check out the [Team Git Cheatsheet](team-git-cheatsheet.pdf).

## Further reading

- [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)
- [5 types of Git workflow that will help you deliver better code](https://buddy.works/blog/5-types-of-git-workflows)

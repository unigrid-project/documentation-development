# Committing
Some general commit and contribution instructions for everybody working at Unigrid or externally contributing code to the project.

## Commit messages and log
It is important that the work in the log can be followed. Please keep your commits short if you can and write messages that
clearly describe what the commit changes or implements.
 * Commit messages should have a max width of 75 characters and follow the following format. Here we marked the right 75
   character column with a pipe character `|`:
   ```
   Add the ability to fart around                                           |
                                                                            |
   If needed, this is a more in-depth description of what the change        |
   briefly covered in the descriptive commit message. We can write as much  |
   as we want here, but ideally it should be a maximum of 7 rows, as this   |
   is something that a lot of git log viewers optimize for.                 |
   ```
 * Make small commits. When you sit and work it's good practice to make continous commits to describe your thought process
   and how you have worked. If someone needs to familiarize themselves with your code, this also makes things a lot easier
   for them. You can use tools such as `git gui` to assemble short and understandable commits.
 * What if you need to clean up your log? Maybe you have commits that you want to merge with another commit. Maybe there
   is a commit message that isn't complete enough or has a typo? For this, `git rebase --interactive` can help you. It
   allows you to interactively make changes to your log, remove commits, change them, squash them and more.

## Branches
When working with branches, there are a number conventions to consider:
   * Keep local branches local. If you need to push it to the common repository, give the branch an appropriate name that
     describes the feature or problem you are working on. If it's a private and temporary branch that is just for you and
     you still need to push it to the repository, prepend it with your initials. If your name is *John Smith*, you would
     name your branch `js-myfeature`. **For the sake of everybody's sanity, PLEASE stick to lower case in branch names.
     In fact - stick to lower case for everything you do in Git if you want to avoid problems under for example Windows!**
   * If it's a branch that you want to push in order to catalogue your work for later use, make it a public branch without
     your initials. This will allow for somebody else to pick up that work later if needed. This is also true for
     collaborative branches that you work on with somebody else - never put your initials on these.
   * Pull requests can use branch names with or without initials prepended.

## Merging branches
When you merge your work, please follow these rules:
   * Clean up your commit log before any merges and make sure it follows the convention in
     [Commit messages and log](#Commit-messages-and-log). That, or keep a clean house when you work so you don't have to
     once it's time to merge.
   * Unless you are interested in preserving branch points, it's generally better to rebase than do a normal merge as it
     keeps the history clean. This will allow us to avoid *Git merge hell*. Use common sense and make a decision.
     Sometimes it is reasonable to keep the branch history. This is especially true whenever it is a long-running
     branch that took, for example, weeks to implement.

     ![image](https://user-images.githubusercontent.com/1555683/220251375-985b7829-df79-438d-81ff-6a3320876e9f.png)

     Not following this rule risks the repository to end up in what's called *Git merge hell* as shown in the image above.
     We definetly want to avoid this scenario.

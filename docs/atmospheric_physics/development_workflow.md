# Development workflow for atmospheric_physics

This page describes the general workflow for adding new developments to the [atmospheric_physics](https://github.com/ESCOMP/atmospheric_physics) repo, which is the location of all CGD-developed CCPP-ized physics schemes and Suite Definition Files (SDFs).

## Workflow summary

The general workflow for adding a feature, bug-fix, or modification to atmospheric_physics is as follows:

  1.  [**Open an issue.**](#1-open-an-issue)
  2.  **[Add your code modifications](#3-update-your-code-with-changes-from-the-official-repo) to a branch on your [fork](#2-create-a-fork-if-you-havent-already).**
  3.  **[Open a PR](#5-creating-a-pull-request-pr) from your branch to the `development` branch.**
  4.  **Respond to any reviewer requests.**
  5.  **Fix any failing tests.**
  6.  **[Update `doc/NamesNotInDictionary.txt` file.](#updating-namesnotindictionarytxt-file)**
  7.  **If you know that this PR will need an official tag, then also add the [tag name](Tagging-Instructions.md/#tag-naming-conventions) to the PR description.**
  8.  **Squash the commits and merge the PR (e.g. the "squash and merge" option).**

If you need an official tag for your new additions, then once your `development` PR has been merged you will need to do the following:

  1.  **Open a PR that merges the atmospheric_physics `development` branch into `main`.  Ensure that the PR description lists every PR that went into `development` since the last update to `main`.**
  2.  **Fix any failing tests.  This includes tests on the target host models that will be using the new tag.**
  3.  **Merge (do not squash!) the PR.**
  4.  **Tag the new merge commit.**

## Workflow details

The following sections describe various workflow actions in more detail.

### 1.  Open an issue.

It is generally recommended to [open an issue](https://github.com/ESCOMP/atmospheric_physics/issues/new) for any new feature that will be added or bug that has been found that will need to be fixed.  There is currently no official requirement on what should be contained within the issue text, so generally just put any information you think might be relevant.

### 2.  Create a fork (if you haven't already).

We recommend creating a fork of the atmospheric_physics repo, and doing all of the development there.  Instructions for how to set up a fork, and how to configure git in general, can be [found here](../development/git-basics.md).

### 3. Update your code with changes from the official repo.

**1.  Clone your repo and add an upstream remote**

```
   git clone https://github.com/<GitHub userid>/atmospheric_physics
   cd atmospheric_physics
   git remote add upstream https://github.com/ESCOMP/atmospheric_physics.git
```

**2.  Fetch the latest version of the ESCOMP repo:**

```
   git fetch upstream
```

**3.  Rebase your local forked branch to the ESCOMP version.  For example, assuming you are working on a local branch called `feature`:**

```
   git checkout feature
   git rebase upstream/development
```

Please note that you may also do a `git merge upstream/development` if you feel more comfortable with that method.

If you then want to update the `feature` branch on your Github fork, then push your changes like so:

rebase:
```
   git push -f
```

merge:
```
   git push
```

Of course, if you run into any problems with either method then please create a discussion post that contains your message and someone will try and assist you.

### 4. Committing code

**1. Create a new branch off of the ESCOMP development branch:**

```
   git checkout -b cool_new_feature upstream/development
```

where `cool_new_feature` is an example branch name (you can use any name you want).  Next, push that new branch to your fork on Github:

```
   git push -u origin cool_new_feature
```

**2.  Apply your code modifications and/or script additions, and perform at least one test making sure that your modifications work as expected.**

**3.  Add all the files that you want to commit.  There are multiple ways to do this, but one of the safer ways is to first check your status:**

```
   git status
```

This will provide a list of all modified files.  For each one of those files whose modifications you want to add to the main package, you will do the following:

```
   git add awesome_scheme.meta
```

Where `awesome_script.meta` should be replaced with whatever your file name is.  Do this for each file you want to include.  If you are confident that every file listed by `git status` needs to be added, then you can do it all at once by doing:

```
   git add -A
```

You can then type `git status` again, at which point all of the files you added should be "staged" for commit.

**4.  Commit your changes to your local branch:**

```
   git commit -m "<message>"
```

where `<message>` is a short descriptor or sentence stating what the commits are for, e.g. "Fixed color bar bug" or "Added significance hatching".

**5.  Push your committed changes to your fork:**

```
   git push
```

### 5. Creating a Pull Request (PR)

**1.  Go to the [ESCOMP atmospheric_physics repo](https://github.com/ESCOMP/atmospheric_physics), and click on the "Pull requests" tab.**

![text](figures/atm_phys_PR_tab_circle.png "Pull requests tab")

**2.  There, you should see a "New pull request" button, which you should click.**

 ![text](figures/new_PR_button.png "New PR button")

**3.  On the new "Compare changes" page, you should see a "compare across forks" link, which you should click.**

 ![text](figures/compare_forks_link.png "Compare forks")

**4.  You should now see two new pull down boxes (to the right of an arrow).  Using those pull down boxes, select the "development" branch of the ESCOMP repo and select your fork (which should be `<username>/atmospheric_physics`):**

 ![text](figures/fork_repo_dropdown.png "fork select list")

  **Then select the branch which contains the new commits:**

 ![text](figures/fork_branch_dropdown.png "branch select list")

**5. You should then see a list of all the different modifications.  If they generally look correct to you, then click the "Create pull request" button.**

 ![text](figures/create_PR_button.png "Create PR button")

**6. A new page should appear.  In the first text box add the title of your Pull request.  The second text box should contain additional fields that you should fill out to the best of your ability.**

**7.  If you are contributing something into development that needs to go into the `main` branch quickly then please make it known in the PR description.  Also add the eventual tag name to the PR description.**

**8.  Add any relevant labels to the Pull request, add yourself as the assignee, and add any reviewers you would like to have.  Otherwise the core SE team will add reviewers for you.**

**9.  Fix any failing tests that show up on Github.**

**10.  If you get any change requests during code review, then simply apply those changes in the same way you applied your original modifications.  Please note that once you `push` your changes then the PR will automatically be updated and the GitHub tests will be automatically run.**

**11.  Update the `NamesNotInDictionary.txt` file using the instructions [below](#updating-namesnotindictionarytxt-file).**

**12.  Once all reviewers sign off on your modifications, then the PR will be squashed and merged.  Congratulations!  Your code is now in atmospheric_physics!**

### Updating NamesNotInDictionary.txt file

**1.  Clone the CCPP standard names dictionary.**

`git clone https://github.com/ESCOMP/CCPPStandardNames.git`

**2. Run the "meta_stdname_check.py" script to generate a new "NamesNotInDictionary.txt" file.**

`CCPPStandardNames/tools/meta_stdname_check.py -m </path/to/atm_phys_repo> -s CCPPStandardNames/standard_names.xml > NamesNotInDictionary.txt`

Where `</path/to/atm_phys_repo>` is a path to the head of your atmospheric_physics repo with all of the relevant changes.

**3.  Replace old names file with new one.**

`cp NamesNotInDictionary.txt </path/to/atm_phys_repo>/doc/NamesNotInDictionary.txt`

Finally, once the `NamesNotInDictionary.txt` file has been updated, then commit and push it to your branch/fork on Github following the instructions in the section above.

### Removing old branches

Once your modifications have been merged into the official CAM repo, you may have no more use for the local fork branch created to develop those modifications.  In that case, you can remove the branch both from your local cloned repo and your atmospheric_physics fork:

**1.   First, make sure your local repo isn't checking out the old branch, by simply checking out a different branch:**
```
    git checkout <some_other_branch>
```
**2.   Then, remove branch from local repo:**
```
    git branch -d <branch_name>
```
**3.   Finally, remove branch from personal fork repo:**
```
    git push --delete <origin> <branch_name>
```
   You can also remove the branch via GitHub's [user interface](https://help.github.com/en/articles/creating-and-deleting-branches-within-your-repository#deleting-a-branch).

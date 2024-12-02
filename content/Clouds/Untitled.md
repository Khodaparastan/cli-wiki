ref : https://ayushirawat.com/github-cli-10-all-you-need-to-know


## **How to download it?**

To install GitHub CLI on your machine, you can refer to the installation guide found on GitHub. To download click the [Link](https://cli.github.com/) Click the link to navigate to the [guide](https://github.com/cli/cli#installation-and-upgrading)

GitHub CLI is available across all platforms, MacOS, Windows and various Linux.

## [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#authentication "Permalink")**Authentication**

After a successful installation, you need to authenticate your Github account.

To authenticate your account you need to use the following command. 

COPY

COPY

```
gh auth login
```

Follow the series of steps to complete your authentication process. After authentication, you can use the GitHub CLI.

# [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#managing-github-repositories-using-github-cli "Permalink")**Managing GitHub Repositories using GitHub CLI** 

`gh repo` command is used to create, clone, fork, and view repositories.

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#1-view-github-repositories "Permalink")**1. View GitHub Repositories**

Display the description and the README of a GitHub repository. With no argument, the repository for the current directory is displayed.

COPY

COPY

```
gh repo view [<repository>] [flags]
```

**Open a repository in the browser**

You can open the repository in a web browser with `-w, --web` commands instead.

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#2-create-github-repositories "Permalink")**2. Create GitHub Repositories**

To create a new GitHub repository use the following command.

COPY

COPY

```
gh repo create [<name>] [flags]
```

for example: create a repository with a specific name

COPY

COPY

```
$ gh repo create demo-repository
```

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#3-fork-github-repositories "Permalink")**3. Fork GitHub Repositories**

Lets fork of a repository with GitHub CLI.

If you don't provide any argument, creates a fork of the current repository. Otherwise, forks the specified repository.

COPY

COPY

```
gh repo fork [<repository>] [flags]
```

# [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#working-with-pull-requests-using-github-cli "Permalink")**Working with Pull Requests using GitHub CLI**

Let’s explore the use of GitHub CLI for managing pull requests.

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#1-list-pull-requests "Permalink")**1. List Pull Requests**

COPY

COPY

```
gh pr list
```

If we want to list out ALL of the pull requests, both open and closed, we could use the “state” flag

COPY

COPY

```
gh pr list --state "all"
gh pr list -s "all"
```

when listing out all PRs, instead of using this full command

COPY

COPY

```
gh pr list --label "labelname"
```

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#2-check-pull-request-status "Permalink")**2. Check Pull Request Status**

If you wish to check the status the PRs you created earlier, you can use the `status command` to list them at the terminal

COPY

COPY

```
gh pr status
```

This will give you a list of PRs that are assigned to you, mentioning you, or that were opened by you.

If you wish to check whether a PR is closed or not, use the following command.

COPY

COPY

```
gh pr list --state "closed"
gh pr list -s "closed"
```

If you wish to list out all of our open bug fix PRs, you could check by listing all the open PRs again

You could also filter by the defining label-name in the GitHub repo.

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#3-view-pull-request "Permalink")**3. View Pull Request**

You can open the PR from the command line using the view command.

COPY

COPY

```
gh pr view [<number> | <url> | <branch>] [flags]
```

It will open the pull request in a web browser where you can then assign it to yourself, review it, etc.

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#4-create-pull-request "Permalink")**4. Create Pull Request**

You can use the following command to create a new pull request directly from the command line.

COPY

COPY

```
gh pr create --title "Pull request title" --body "Pull request body"
```

You’ll have an option to submit the PR, you can open it the browser, or cancel.

If you prefer to create your PR from the web, you could use the following command to open up a browser window.

COPY

COPY

```
gh pr create --web
```

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#5-checkout-pull-request "Permalink")**5. Checkout Pull Request**

You can checkout a Pull Request in GitHub CLI using the following command.

COPY

COPY

```
gh pr checkout {<number> | <url> | <branch>} [flags]
```

# [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#managing-github-issues-using-github-cli "Permalink")**Managing GitHub Issues using GitHub CLI** 

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#1-list-issues-with-github-cli "Permalink")**1. List Issues With GitHub CLI**

COPY

COPY

```
gh issue list
```

If we want to list out ALL of the issues we could use the `state` flag

COPY

COPY

```
gh issue list --state "all"
gh issue list -s "all"
```

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#2-checking-issue-status-with-github-cli "Permalink")**2. Checking Issue Status With GitHub CLI**

You can use the following command to list them at the terminal.

COPY

COPY

```
gh issue status
```

This will give you a list of issues that are assigned to you, mentioning you, or that were opened by you.

In case you cannot find the issue you’re looking for, so let's check if its closed.

COPY

COPY

```
gh issue list --state "closed"
gh issue list -s "closed"
```

If you wish to filter out you can filter by the “labelname” label defined in your GitHub repo

COPY

COPY

```
gh issue list --label "labelname"
gh issue list -l "labelname"
```

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#3-viewing-issues-with-github-cli "Permalink")**3. Viewing Issues With GitHub CLI**

You can open the issue from the command line using the following command.

COPY

COPY

```
gh issue view {<number> | <url>} [flags]
```

for example:

COPY

COPY

```
gh issue view "10"
```

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#4-creating-issues-with-github-cli "Permalink")**4. Creating Issues With GitHub CLI**

You can create an issue with the help of the following command.

COPY

COPY

```
gh issue create --title "Issue title" --body "Issue body"
```

At the end you’ll have an option to submit the issue, open it the browser, or cancel.

If you still prefer to create your issue from the web, you could use the following command

COPY

COPY

```
gh issue create --web
```

# [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#working-with-github-gist-using-github-cli "Permalink")**Working with GitHub gist using GitHub CLI** 

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#1-list-gist-with-github-cli "Permalink")**1. List gist with GitHub CLI**

You can list your gists with GitHub CLI using the following command.

COPY

COPY

```
gh gist list [flags]
```

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#2-view-gist-with-github-cli "Permalink")**2. View gist with GitHub CLI**

You can view your gists with GitHub CLI using the following command.

COPY

COPY

```
gh gist view {<gist id> | <gist url>} [flags]
```

### [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#3-create-gist-with-github-cli "Permalink")**3. Create gist with GitHub CLI**

Create a new GitHub gist with given contents. Gists can be created from one or multiple files. Alternatively, pass “-“ as file name to read from standard input.

By default, gists are private; use ‘–public’ to make publicly listed ones.

COPY

COPY

```
gh gist create [<filename>... | -] [flags]
```

# [](https://ayushirawat.com/github-cli-10-all-you-need-to-know#working-with-github-alias-using-github-cli "Permalink")**Working with GitHub alias using GitHub CLI** 

You can print out all of the aliases `gh` is configured to use using the following command.

COPY

COPY

```
gh alias list [flags]
```
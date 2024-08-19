---
layout: post
title: 'Git: Keeping your email safe in public repositories'
date: 2024-08-18 18:51:44 -0500
categories: posts git
tags: Tools,Git
---

In today’s digital age, it’s easier than ever to share your code and contribute to public projects, often as a way to showcase your portfolio. However, the open nature of these platforms can sometimes lead to accidentally sharing personal information, like your email address. Here’s a simple guide on how to keep your personal information safe while still sharing your work publicly.

1. <b>Use Dedicated Email Address:</b> Create an email address specifically for your GitHub account or other open source activities. This way, even if your email address is exposed, your personal email remains secure.

   configure your git settings to use this email address:

   ```bash
   git config --global user.email "dedicated-email@example.com"
   ```

2. <b>Use github's Private Email Feature:</b> Github provides a feature for developers to keep their email address private for commits.
   Here's how you can do it.

   - Navigate to <a href="https://github.com/settings/emails" target="_blank">https://github.com/settings/emails</a>
   - Add Email Address that you might not want to expose.
   - Check the `Keep my email addresses private` option.
     In the description section you can find your `username@users.noreply.github.com` email which will be used for web based activities.
   - Check `Block command line pushes that expose my email` option.
     While performing push operations, github will check if the latest commit has your private email, if yes the operation will be blocked
     and git will warn you about the exposing of the private email.

     ![github warning for exposing private email](/assets/images/keeping-email-private/warn-private-email-exposure.png)

   - Configure your local git settings to use your github noreply email.

     ```bash
         git config --global user.email "username@users.noreply.github.com"
     ```

3. <b> what if your email is already exposed ? </b>

   > **_WARNING ⚠️ :_** This option will rewrite your git history, hash values of the commits, so use carefully.

   - Install <a href="https://github.com/newren/git-filter-repo" target="_blank">`git-filter-repo`</a>: tool used to rewrite the git repository history.
   - `git log --all --format='%h %ad %an <%ae> %cn <%ce>’` to check which email addresses are exposed in commits.
   - use the below command to rewrite your commit history and replace your exposed email.

     ```bash
         git filter-repo --commit-callback '
         if commit.author_email == b"your_exposed_email":
         commit.author_email = b"github-email@users.noreply.github.com"
         if commit.committer_email == b"your_exposed_email":
         commit.committer_email = b"github-email@users.noreply.github.com"
         '
     ```

     > This might remove your remote references, so verify it using `git remote -v` if removed, add the remote references back.

   - confirm the changes with `git log --all --format='%h %ad %an <%ae> %cn <%ce>’`.

   - since the above command rewrites the git history, you need to force push these changes.

     > If the repository is big, this push can give you error because of the limited buffer size for http operations.
     > use below command to increase your git http buffer size.

     ```bash
         git config --global http.postBuffer 157286400 # ~150MB
     ```

It is best to avoid third option, Since you are mostly working in groups in open source projects and it is not a wise to completely rewrite the git history. So,

1. Avoid hardcoding personal information.
2. Educate yourself About open source best practices.
3. Regularly review your repositories.

And be mindful of the data you share.

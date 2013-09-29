Gitolite hooking system + hooks
===

This is a very simple system for adding post receive hooks to your gitolite repositories.  It was first escribed in [A hooking system for Gitolite](http://demonastery.org/2012/09/a-hooking-system-for-gitolite/).

License
---

Consider the scripts here to be in the public domain. However any contributions will be highly valued!

Installation
---

### Prepare Gitolite

_Note. I am assuming Gitolite v3 for this, but it may be similar for v2 with some tweaking._

This hooking system exists almost entirely inside the Gitolite admin repository, but for that to work we first need to make some changes to .gitolite.rc over on the server side. So go ahead and open that up and make the following changes:

* Change the `LOCAL_CODE` directive to `"$ENV{HOME}/.gitolite/local-code"` (this allows our hook to fire)
* Change the `GIT_CONFIG_KEYS` directive to `".*"` (this is to be able to use hooks.run later on)

Save the file and run `gitolite setup` at the shell to apply these changes. 

### Add Gitolite Hooks to the Gitolite Repo

Add Gitolite Hooks to your existing Gitolite admin repository by adding the local-code directory tree to the root of the admin repo.

    cd gitolite-admin
    git clone https://github.com/ZaneA/Gitolite-Hooks.git
    cp -a Gitolite-Hooks/local-code .
    rm -rf Gitolite-Hooks

Using Hooks on a Repo
---

To specify a post-receive hook to be run for a repository, edit the gitolite.conf to add a `config hooks.run = _hookName_` entry for that repository.  This example run the  `nginx-deploy` and `notify` hooks each time one of the web/..* repos receives an update.

	repo    web/..*
	  C = @zanea
	  RW+ = @zanea
	  config hooks.run = nginx-deploy notify

You can also specify hooks on a single line in a .hooks file inside the repository itself (useful for wild repos).  

Defining Hooks
---

The hooks you reference in hooks.run must be available in `local-code/hooks/common/hooks.d`.  You can write these to suit your environment.  The git hooking mechanism allows any program to be run as a post receive hook.  The hook has these environment variables available to it at run time among others.

* oldrev - commit ID of the previous position of HEAD
* newrev - commit ID of the new position of HEAD
* refname - refname being updated
* GL_USER - remote user that pushed to the repo
* GL_REPO - name of the repo

Gitolite 3.5.2 introduced a method to set arbitrary environment variables on a per repo basis.  These variables are accessible to the hook.  For more complete information on this and other developer features see the [Gitolite - notes for developers](http://gitolite.com/gitolite/dev-notes.html)  

Several example hooks are described below.  


Available Hooks
---

These hooks are examples written in bash.  Most of these will probably need some editing to work with your setup, so make sure to check out the code before blindly running them.

### test-print

Prints out some information for debugging: Working directory, $GIT_DIR, $oldrev, $newrev, $refname.

### update-redmine-repositories

Triggers your local Redmine instance to update all configured repository changesets. More info at http://demonastery.org/2012/10/a-redmine-hook/.

### generate-documentation

Builds project documentation to /srv/documentation/PROJECT-NAME/ using Doxygen.

### checkout

Checkout HEAD of the repository into /tmp/git/PROJECT-NAME/ for further manipulation.

### jekyll

Use Bundler and Jekyll to build /tmp/git/PROJECT-NAME/jekylldocs/ to /tmp/git/PROJECT-NAME/htdocs/. Requires `checkout` hook to be specified before this one. Hook will not run if /tmp/git/PROJECT-NAME/jekylldocs/_config.yml is missing, so is safe to run on all web repositories.

### nginx-deploy

Copies a checked out and built project from /tmp/git/PROJECT-NAME/ to /www/PROJECT-NAME. Will also symlink and reload Nginx configuration.

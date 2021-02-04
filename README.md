# wptheme


## What's important?

### True independent versioning

Some teams may need to hold back on upgrades. Especially true of the design 
system. Say a change occurs as needed by the WebApp, but it breaks the 
Wordpress theme. 

- Option: **Include packages as `npm` package dependencies.** Puts dependent
  code in the `node_modules` folder. Provides no method of automatic updates. 
  Will need to manually check to see if we have the latest version, unless we
  use SVN ranges (like `^1.0.0`). Lerna doesn't support this unless the package
  is published to NPM. So... we'd need to manually the `package-lock.json` 
  in each package folder, or else it will install the previous version.
- Option: **Independent versioned Lerna packages.** Doesn't acutally support this.
  Despite each package having it's own version number, the only 'bundleable' release
  can only include the currently versioned instances. It will not independently 
  symlink versions of pacakges in the same monorepo. 
- Option: **CLI support.** Provide CLI tools to install **any** desired version
  and build with that version for Spreadless packages. Support symlinking 
  into the `/packages` folder if needed for builds. Need to track which version
  of dependent pacakges the project is built with. Need to support "dirty" 
  sub-packages similar to `lerna diff`. None of this is terribly difficult using
  octokit, but auth becomes a PITA to use globally. Need Oauth **and** SSH, or
  would need to do everything through the API. Neither of these are appealing. 
- Option: **Publish packages to private npm or git repo.** This would sovle the
  versioning issue, but adds publishing steps, and makes peer packages difficult
  to edit/update. Would allow us to use Lerna out of the box. 
- Option: **Use git submodules**. After some experimentation, these do much of 
  what we want. They are independently versionable. If local changes are made,
  they "dirty" up a git repo, and can be independtly pushed to the respective
  repo. My only pause... it's damn easy to miss the dirty submodule in a UI. 
  Fork shows an `*` but that's it. New versions created outside... aren't even
  shown uless you click into the submodule, `fetch` and `checkout` the tag
  desired. The CLI experience is... worser. It's terrible. 
- Option: **Symlinked distribution folders**. 100% Guaranteed workable software, 
  sorta. Let's call it best chance rather than 100%. This case each package
  builds into a folder (in that package's tree). Dependent packages symlink 
  the outputs. "Builds" could be complex (like for a design system) or simple,
  maybe even simple file copy for shared components. The dependent package's
  symlinked folder... is under SCM. It gets published as-is in the as-is state
  when a release is created. Could the user release bundled code that is unpushed?
  Yep. But we can provide some CLI support to help with that. 

  The local changes "dirty" up the package repo as we'd want. The source code is accessable in the
  dependent package (not buried in `node_modules`). How to handle releases? 
  Our dependency graph for Spreadless packages should be small enough, and static
  enough to maintain. So for example if `WebApp` depends on `DesignSystem`, 
  the CLI command `version WebApp --patch` can simply check the Git status
  of `DesignSystem`. The `DesignSystem` can be versioned independently. Say
  the design team releases version `5.0.0`. But `WebApp` isn't ready for the 
  breaking changes. Use the CLI, and do something like 
  `install designsystem --4.8.3`. How to know `DesignSystem` depends on `4.8.3`
  instead of `latest` or `5.0.0`? Add tag in `DesignSystem`'s `package.json` 
  file. The CLI command `install webapp` then checks and enquires which version
  to install locally. If the git status is dirty... it won't install 
  *anything* or requires `--force` to overwrite any local changes. 
  
  How do we handle builds? Dependency graph! If `WebApp` requires `DesignSystem`, 
  build `DesignSystem` first. Maybe standardize to a `tasks/build.js` folder?
  Call it async, sync, or only on changes as needed. The dependent package can
  deine that in it's own build. 

The last option is clearly the most bespoke. Yet, it also leaves a decent CLI
experience for pushing / fetching changes from git remotes. It's like a 
mini-package manager. Am I duplicating what `npm` or `lerna` could do? Not 
really. They don't handle versionsing nor symlinking as I'd like. Lerna has
hard requirement for simultaneous releases--but I **specifically** do not want 
that restriction. I want to operate my business as small independent teams 
(I think!) Requiring 3-5 teams to simultaneously release seems... silly. And 
requireing teams progressing on an unrelated problem to: STOP, and test another
team's release seems even stupider. 

The monorepo approach damn near requires trunk based development. Because one 
team's changes could break antoher team's code... testing must be... 100% 
rigorous. 

One last thought to consider. Using git submodules and symlinked files from 
poly repo are 100% compatable. Forward and backward. *what is the least 
amount of code I could write?* Git submodles requires zero code. But there's
a good chance we'll hate the workflow. If we do... we have a **version 2.0**
option already in the pocket, ready to go. Hmm.....

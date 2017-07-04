# A Branching Strategy for a Unity Project in Public Alpha, Using SVN

## Introduction

Metamorph is a Unity project that has been under active development for more than two years, in the hands of a wide cast of developers. During that time, changes to the code and assets have been stored and shared in a SVN repository. This approach has been broadly successful for in-house development and producing builds to demo at game industry shows, but with a Steam Early Access release planned for September 2017, we will shortly be required to maintain at least two different versions of the game: a stable (or, least, broadly playable) one that is available via Steam to paying customers, and a potentially unstable, buggy one with new, untested features that is only visible within Firefly. This increased complexity of production requirements will in turn require more involved strategies to manage changes to the project.

This document describes a strategy for managing project changes using SVN branches to split sets of changes into separate groups that can be merged selectively to produce builds of varying expected stability levels.

### The Problem

Once Metamorph has been released into Early Access and there is a live version in the hands of players (who will be keen to receive regular updates, but understandably upset if we break the game they've paid for), development will have to satisfy multiple competing aims. Specifically, we will want:
 - to show to early-access users that we are making active progress on the game by releasing frequently and regularly (possibly even weekly).
 - QA to be able to test a candidate version of the game before it is released, to avoid shipping a version of the game that breaks features that players were already enjoying.
 - to be able to continue work on new features without disrupting QA's testing of the candidate build.
 - to be able to work on complex features that may require development times longer than the time between releases.
 - to be able to issue hotfixes for critical bugs in the currently-released build, and have those fixes survive into future releases.
 - to be able to patch bugs discovered in the candidate version and have those fixes survive into future releases.

## Branches and tags

Branching is a feature available in most version-control systems that allows for multiple versions of the code and assets to exist side-by-side in the repository. This means that changes made to one branch do not affect others, so untested or work-in-progress commits can be made in one area of the project while preserving a known-good branch elsewhere in the repository. For a broad introduction to the concept, I'd recommend [Jeff Atwood's blog post][1]<sup>1</sup> describing branches in version control by using the idea of "parallel universes" as an analogy.

Tags are a similar concept to branches, but they are intended to be immutable. That is, once you have created a parallel version of your code by tagging it, you will never commit any further changes to the tagged version, meaning it continues to exist in its original state (with bugs and all) forever. The main use for tags is to be able to refer back to specific named versions of the code at a future date. For example, you might tag the revision of your code that was released as v1.3 as "1.3" so that, even after releasing 1.4 or 1.5, you can still easily jump back to the state of the code in 1.3 to track down bugs reported in the older version (or to find out why something that was working then isn't any more...).

### Using branches in SVN

SVN treats branches as "copies" of the original code that exist within a different subdirectory of the repository. The typical directory structure for an SVN project is therefore:

 - `/trunk` - Contains the "trunk", or "master" branch of the project.
 - `/branches` - Contains several subdirectories, each holding a branch of the project.
 - `/tags` - Contains several subdirectories, each holding a tagged version of the project.

Metamorph, however, was not started with branching in mind, so the repository root instead contains several directories containing things like build server output, art assets and design docs. The Unity project itself lives within the `/Project` directory. Fortunately, because SVN just treats branches as directories, we can just store them anywhere we like in the repo. I've already created a `/project-branches` directory in the root which can be used to hold branches of the `/Project` directory. I suggest we also add a `/project-tags` directory to hold tagged versions in future.

#### Creating branches

Creating a branch in svn is easy. You can use the `svn copy` command on the command-line, specifying the remote location you want to branch from, the location you'd like to create the new branch, and a commit message describing what you're doing:

```
> svn copy https://svn.server.url/repo-location/Project \
           https://svn.server.url/repo-location/project-branches/b ranch-name \
           -m "Created 'branch-name' branch to hold some changes."

Committing transaction...
Committed revision 6448.
```

Alternatively, you can use the "Branch/tag" option in the Tortoise SVN GUI:

![Create a branch in Tortoise SVN](tortoise-create-branch.png)

#### Switching branches

Usually after creating a branch, your local working copy will still be working on whatever branch it was on before (although there is a "Switch working copy to new branch/tag" option in the Tortoise SVN GUI when creating a branch, visible above).

### Using branches in Unity

 - Use prefabs to break scene components into separately-editable chunks.
 - Don't fear the merge. They're only YAML, so it might be okay!
 - It isn't always, though, so use a "social" locking system. This is far from perfect, but it's probably the lowest-overhead solution for a small team.

## The Strategy

The strategy described here and the images used to illustrate it are stolen more-or-less wholesale from [Vincent Driessen's excellent 2010 blog post][2]<sup>2</sup> about a branching strategy he was using for git. The only changes I have made are to describe its use with SVN instead of git.

![Branching overview](overview.png)

### Main branches

![Main branches](main-branches.png)

### Release branches

### Feature branches

![Feature branches](feature-branches.png)

### Hotfix branches

![Hotfix branches](hotfix-branches.png)

## References

1. [Software branching and parallel universes][1], Jeff Atwood, _Coding Horror_, Oct 2007
1. [A successful git branching model][2], Vincent Driessen, _nvie.com_, January 2010
1. [Project organization is no monster][3], Bruno Croci, _Made With Unity_, January 2017
1. [Version control: Effective use, issues and thoughts, from a gamedev perspective][4], Ash Davis, _Gamasutra_, Nov 2016
1. [Chapter 4. Branching and Merging][5], Ben Collins-Sussman, Brian W. Fitzpatrick & C. Michael Pilato, _Version Control with Subversion (for Subversion 1.7)_, 2013

[1]: https://blog.codinghorror.com/software-branching-and-parallel-universes/
[2]: http://nvie.com/posts/a-successful-git-branching-model/
[3]: https://madewith.unity.com/en/stories/project-organization-is-no-monster
[4]: http://www.gamasutra.com/blogs/AshDavis/20161011/283058/Version_control_Effective_use_issues_and_thoughts_from_a_gamedev_perspective.php#usingversioncontroleffectively
[5]: http://svnbook.red-bean.com/en/1.7/svn.branchmerge.using.html

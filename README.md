# xbpskeeper

Declarative package management for xbps. Useful with Void Linux and other xbps-based distros.

## Why use this?

It can be hard to remember why I've installed all the packages on my system. This can cause various issues:

* __My system is full of crufty packages that I don't really need__. For example, if I temporarily install some packages to compile something, I'll probably never remove them later.
* __It's easy to accidentally remove important packages__. For example, if I install `ruby-pry`, that will pull in `ruby`, and mark it as automatically installed. Later if I decide I don't want ruby-pry, an `xbps-remove -o` will get rid of ruby, even if I'm using it for lots of other things!
* __It's hard to setup my packages on a new system__. I can't just look at my previous system, cuz I have no idea what all its packages were for.
* __I can't tell what I used to have in the past__. If I was previously developing on some project, I probably installed a bunch of packages, and maybe removed them later. If I want to get back to that project, I doubt I can remember exactly which packages I used.

## How to use this

* Make sure you have our only dependency installed, the package `ruby`.
* Copy `xbpskeeper` somewhere in your PATH, for example ~/bin or /usr/local/bin.
* Create a directory ~/.config/xbpskeeper to declare your packages
  * Inside that directory, create one or more files with the extension `.keep`.
  * Each file should contain a list of packages you want to keep installed. Comments starting with '#' are allowed, as are blank lines.
  * You can get a reasonable initial list of packages with `xbpskeeper candidates`
* Once you think you're satisfied, run `xbpskeeper diff`. It will tell you what it thinks needs removing or installing.
  * Then, run `xbpskeeper sync` to do the installation or removal! It will take care of installing and removing dependencies.
* Later, when you want to make changes, just edit your .keep files, then run `xbpskeeper sync`

An example .keep file might look like:
```
# Browsers
firefox
chromium
epiphany # aka "Gnome Web"

ripgrep tmux git # CLI tools
```

## Tips

* Put your xbpskeeper directory in git! Then you'll have a history of your changes.
* Use comments to remind yourself why you wanted a package, so you know whether you still need it.
* Need a single package, and too lazy to open your editor? Just run `xbpskeeper add foo "my reason" mypackage`, and it will automatically put it in `foo.keep` and install it. Then you can organize it properly later, when you have more time.
* Put packages required for a project in a separate .keep file, so you remember that they're related. Now they're easy to remove all at once!
   * Similarly, you can put related packages nearby each other, so they're easy to block-comment and get rid of.
* There's no harm in listing a package twice! Eg: if it's needed for multiple projects, you can list it in multiple .keep files.
* Copy your whole xbpskeeper dir to a new system, to setup packages just like on the old one.
* Feel free to install some packages temporarily with xbps-install, without using xbpskeeper, as long as it's safe for them to be autoremoved later.

## Detailed usage

### Common commands

#### xbpskeeper sync

Install and remove packages to match the keeper files

#### xbpskeeper diff

Print the packages that would be installed or removed by a sync

#### xbpskeeper add BASENAME REASON PACKAGE [PACKAGE...]

Adds packages to BASENAME.keep with a comment, and installs them. Good for small changes when you're too busy to open an editor.

#### xbpskeeper candidates

List packages not listed in .keep, and with no dependencies. Good for finding candidates to add to .keep files, especially when starting out.

### Other commands

keepers: List packages in .keep files
missing: List packages to be installed
unneeded: List packages to be removed
install: Install missing packages (but don't remove anything)
remove: Remove unneeded packages (but don't install anything)
update-reasons: Unconditionally update xbps package reasons, according to the keepers

## Why not use...?

### xbps-pkgdb

xbps already has a built-in concept of marking package installation reasons as "manual" or "auto". You can use the `xbps-pkgdb` command to manipulate this, and `xbps-remove -o` will cleanup packages that are no longer needed.

But:
* xbps-pkgdb has no user-visible keeper file at all!
  * You can't put your keepers in git.
  * You can't add comments or split things into multiple files.
* By default, xbps assumes that when you install a package, it's desired as a keeper. This makes it too easy to install packages without remembering why.

### pacdef

[pacdef](https://github.com/steven-omaha/pacdef) is a multi-backend declarative package management system. You can manage packages across a variety of different distros and language-specific package managers, like cargo or pip. It supports a hierarchical system of package grouping, where entire groups of packages can be imported or removed from your system.

But:
* There are [serious bugs](https://github.com/steven-omaha/pacdef/issues/90) that make pacdef very difficult to compile.
* It's much more complex than I need.
* It doesn't support certain use cases I like, such as commenting out a whole file when I'm done with a project.

### ansible, chef, puppet

These are great tools to set up systems. But they're designed for production systems that should mostly stay unchanged. End-user systems constantly have packages being installed and removed, and are a better fit for something like xbpskeeper.

## Caveats

* Your .keep files are __not__ a full description of the packages installed on your system. xbps dependencies can be optional, or can have multiple packages that satisfy a given dependency, and there's no way for xbpskeeper to understand your intentions. Typically it works, but if you need to have absolutely identical packages on different systems, use a different solution.

## Inspiration

* [aptkeeper](https://github.com/vasi/aptkeeper), my nearly equivalent system for apt-based distros.

* [debfoster](https://packages.debian.org/sid/debfoster) for apt-based distros (Debian, Ubuntu, etc)
* [pacdef](https://github.com/steven-omaha/pacdef) for Arch-based distros
* [pacmanfile](https://github.com/cloudlena/pacmanfile), also for Arch-based distros
* Homebrew's [bundle](https://github.com/Homebrew/homebrew-bundle) capabilities.
* The [set -A](https://man.freebsd.org/cgi/man.cgi?query=pkg-set&sektion=8&apropos=0&manpath=FreeBSD+14.2-RELEASE+and+Ports) and [autoremove](https://man.freebsd.org/cgi/man.cgi?query=pkg-autoremove&sektion=8&apropos=0&manpath=FreeBSD+14.2-RELEASE+and+Ports) subcommands of FreeBSD's pkg
* The [selected set](https://wiki.gentoo.org/wiki/Selected_set_(Portage)) of Gentoo's Portage, along with the `--select`, `--deselect` and `--depclean` options.
* The [mark](https://dnf.readthedocs.io/en/latest/command_ref.html#mark-command-label) and [autoremove](https://dnf.readthedocs.io/en/latest/command_ref.html#autoremove-command-label) subcommands of Fedora's dnf
* Of course, the declarative systems of [Nix](https://nixos.org/) and [Guix](https://guix.gnu.org/)

Also some previous attempts of mine at this sort of thing:
* [brewfoster](https://github.com/vasi/brewfoster) for Homebrew
* [rpmkeeper and yumkeeper](https://github.com/vasi/rpmkeeper) for yum-based distros
* [zyppkeeper](https://github.com/vasi/zyppkeeper/) for OpenSUSE's zypper
* [dnf5keeper](https://github.com/vasi/dnf5keeper) for Fedora's DNF5

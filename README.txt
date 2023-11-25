=== SUMMARY ===

Brings package management with Zypper into the 21st Century by providing an
autoremove command as well as facilities for marking package installation
reasons as automatic or manual.

=== VERSION ===

1.1.1

=== MOTIVATION ===

I use Fedora as my daily driver but for various reasons was tempted to switch to
openSUSE. One of the few things stopping me was Zypper lacking the above
mentioned functionality. Hence I built this to engineer a potential future
migration and make life easier for others in the (open)SUSE ecosystem.

It's absolutely absurd that Zypper doesn't have any easy semblance of autoremove
functionality. Best case scenario persistent storage space is wasted. As can be
network bandwidth when updates for no longer needed packages are pulled in.
Worst case scenario vulnerabilities in stray setuid executables left over on
the system can be exploited for privilege escalation.

It's also unfortunate that Zypper doesn't let users mark packages as manually or
automatically installed as is possible with many other packaging systems.

With zypper-unjammed those problems are history.

=== USAGE ===

For marking the statuses of packages run commands like:
zypper-unjammed mark-manually-installed <package> <names> <here>
zypper-unjammed mark-automatically-installed <package> <names> <here>

For autoremove functionality the command variant you most likely want is:
zypper-unjammed autoremove

In rare cases you may need to repeat the autoremove command several times until
it tells you there's nothing for it to do. Usually one invocation will be
sufficient, but you may want to check just in case.

In other rare cases users have experienced the default autoremove command is
overly aggressive and proposes to remove packages it shouldn't. One way to
mitigate this is to mark such packages as manually installed. Another is to
repeatedly use:
zypper-unjammed conservative-depth-first-autoremove
zypper-unjammed conservative-breadth-first-autoremove

The latter is the safer of the two. It leverages "zypper remove --no-clean-deps"
repeatedly to peel back layers allegedly unneeded packages one at a time. The
former uses "zypper remove --clean-deps" to try clean out the dependency tree of
just one allegedly unneeded package.

Always CAREFULLY CHECK any actions Zypper and/or this tool proposes before
saying yes to anything. If you say no to a conservative-depth-first-autoremove
command next time you run it it will randomly pick another likely distinct
allegedly unneeded package whose dependency tree it will offer to clean if you
approve. In rare cases you may need/want to use hybrids of these techniques.

=== LICENSE ===

This is a Freedom Respecting Technology: think Open Source but not only for well
off people with reliable Internet access. Learn more at:
https://makesourcenotcode.github.io/freedom_respecting_technology.html

It's (admittedly small) Open Knowledge Set consists of:
* the main program source/executable file zypper-unjammed
  licensed under GNU GPL v3 ( https://www.gnu.org/licenses/gpl-3.0.txt )
* the documentation / Help Information Set consisting of this file
  licensed under GNU FDL v1.3 ( https://www.gnu.org/licenses/fdl-1.3.txt )

=== DEVELOPMENT NOTES ===

Development and testing was done in an OCI container based off the image:
registry.opensuse.org/opensuse/tumbleweed:latest

The testing process was manual but extensive. Each of the subcommands was tested
multiple times and is robust to issues like variations in the output of various
Zypper commands and users trying nonsense like marking the states of uninstalled
or nonexistent packages. I do wish the tests were more automated.

=== LIMITATIONS AND POSSIBLE FUTURE IMPROVEMENTS ===

Pull requests and/or patches addressing these are welcomed.

1:

Overall the tool does exactly what it says on the tin. Unfortunately right now
there is a small risk of race conditions due to the fact that we don't
lock/synchronize our accesses to /var/lib/zypp/AutoInstalled which other zypper
processes like cron jobs doing automated updates, periodic checks by graphical
software centers for updates, etc may be doing.

Obviously zypper-unjammed is written to do things like batch its
reads/writes thus minimizing vulnerable windows for race conditions. The risk
though quite small is still there so be aware of it. To eliminate the risk make
sure no other zypper processes are running or can run in the background while
zypper-unjammed is running.

Other solutions we've seen from people trying to solve some or all of these
problems in a manual or automated fashion have similar issues as it seems to be
hard to hook into whatever mechanism zypper uses to make sure that there's only
one potentially state changing process running. Not to mention using tools like
flock correctly/safely in a shell script is generally a losing battle. We'd have
written this in literally any other scripting language if we could guarantee its
presence on the system.

2:

It'd be nice to add tab completions for subcommand names and package
names at some point but it's not a high priority as doing this is rather shell
specific and has limited portability.

3:

In the AutoInstalled file there is a comment Zypper leaves indicating the time
it was last modified. Our tool doesn't modify this comment to leave a correct
modification time and hence can be misleading to those looking at it for
whatever troubleshooting purposes. This can easily be worked around by looking
at the modification time of the file at the filesystem level.

4:

Not only does Zypper not have autoremove functionality but it sometimes doesn't
even properly support recursive removal of a given package's dependencies not
needed by other packages. As a fun experiment you may want to grab the official
openSUSE OCI image I used for development, spin up a container from it, then run
"zypper install leafpad && zypper remove --clean-deps leafpad" and notice that
the remove command does not offer to remove all the packages that were just
installed even in spite of --clean-deps being passed. Given that leafpad's
dependencies shouldn't have changed and that there were no other packages
installed in the interim which share dependencies with leafpad this is clearly
erroneous.

While we do a reasonably good job of attaining this tools espoused goals sadly
there are limits to what can be built atop literal quicksand.

=== CHANGELOG ===

v1.1.1: minor documentation fix

v1.1.0: added support for conservative autoremoval variants

v1.0.0: implemented base functionality

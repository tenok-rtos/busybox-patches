# BusyBox patches for Tenok

The changes BusyBox needs in order to be built into the firmware of Tenok.

The submodule that holds BusyBox itself stays exactly as its release left it;
everything Tenok changes lives here and is applied on top by
`scripts/busybox-prepare.sh` of the Tenok tree.

## Why these are not in the Tenok tree

A patch carries the surrounding lines of the file it changes, so these files
contain BusyBox source and are covered by its licence, the GNU General Public
License version 2. Tenok itself is under the BSD 2-Clause licence, and keeping
the two apart is what lets it stay that way.

## The series

Patches are applied in order and each one owns the files it names: the series
is taken against the release commit rather than against the patch before it,
so two patches naming the same file will not both apply.

| Patch | What it changes |
| --- | --- |
| `0001-run-the-applets-without-fork.patch` | Tenok has neither `fork()` nor `exec()`, so an applet can only be reached in process. Every applet is therefore NOFORK, and `run_nofork_applet()` takes back the memory and the descriptors it leaves behind, which a process would have returned on exit |
| `0002-declare-what-tenok-provides.patch` | Tells BusyBox that Tenok does not understand `%m`, and has neither `/dev/fd`, `wait3()` nor `clearenv()`. BusyBox has a `clearenv()` of its own to fall back on |

## Version

Against BusyBox `1_38_0`.

# Patches for the packages Tenok builds

The changes each package needs in order to be built into the firmware of Tenok.
The submodule that holds a package stays exactly as its release left it;
everything Tenok changes lives here and is applied on top by
`scripts/package-prepare.sh` of the Tenok tree.

## Why these are not in the Tenok tree

A patch carries the surrounding lines of the file it changes, so these files
contain the source of the package they are against and are covered by its
licence: the GNU General Public License version 2 for BusyBox, and the GNU
Lesser General Public License version 2.1 for DirectFB2. Tenok itself is under
the BSD 2-Clause licence, and keeping them apart is what lets it stay that way.

## Layout

Every package has a directory of its own, and the patches of that package are
in the `patches` directory inside it. They are applied in order and each one
owns the files it names: a series is taken against the release commit rather
than against the patch before it, so two patches naming the same file will not
both apply.

    <package>/patches/*.patch

## busybox

| Patch | What it changes |
| --- | --- |
| `0001-run-the-applets-without-fork.patch` | Tenok has neither `fork()` nor `exec()`, so an applet can only be reached in process. Every applet is therefore NOFORK, and `run_nofork_applet()` takes back the memory and the descriptors it leaves behind, which a process would have returned on exit |
| `0002-declare-what-tenok-provides.patch` | Tells BusyBox that Tenok does not understand `%m`, and has neither `/dev/fd`, `wait3()` nor `clearenv()`. BusyBox has a `clearenv()` of its own to fall back on |

Against BusyBox `1_38_0`.

## directfb2

DirectFB2 is built for Tenok through the path it already has for NuttX: the
`nuttxfb` system module, the `lib/direct/os/nuttx` operating system layer, and
the source list its `Makefile` names.

Everything here is the same kind of change. The NuttX headers pull in
`<string.h>`, `<unistd.h>`, `<errno.h>`, `<limits.h>` and `<dirent.h>` for the
files that use them, so those files never had to ask. On a system whose
headers do not, they have to.

```
0001  lib/direct/os/nuttx/types.h        the OS layer brings in the platform's
                                         headers, so the ones that were missing
                                         go here and cover most of the tree.
                                         u8 through u64 are given the names
                                         <stdint.h> gives them, so that a
                                         library naming them the same way
                                         agrees with this one about what they
                                         are: uint32_t here is long unsigned
                                         int, and unsigned int is a different
                                         type of the same width. The signed
                                         ones keep the built-in names, which
                                         is what the rest of DirectFB2 passes
                                         around
0002  the headers that use errno, INT_MIN, DIR and struct dirent
0003  the sources that use atexit, getenv, rand, errno and the file calls
```

Nothing else in DirectFB2 is changed. The version these apply to is
`7d4682d` (`Merge pull request .../160`).

Two things are settled in the build rather than here:

- `CONFIG_TASK_NAME_SIZE`, which NuttX takes from its own configuration.
  Tenok says the same thing in `<sys/prctl.h>` as `PR_NAME_MAX`.
- `SIZEOF_LONG`, which the upstream Makefile writes as 8. It is 4 here.

`src/core/Core*.{c,h}` are not in the repository; they are written once as
interface descriptions and turned into C by `fluxcomp`, which
`scripts/directfb-fluxcomp.sh` builds from `github.com/deniskropp/flux`.

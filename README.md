# Database of RPM Macro Definitions / Values

This repository provides **JSON database files** acceptable by [norpm
tooling](https://github.com/praiskup/norpm) for specific macro definition
overrides.

Take this example: consider you're on a **Fedora 43** host, where `%fedora ==
43`, but your build is going to be done for **RHEL 10**.  Naturally, you'd want
the `%fedora` macro to be undefined, and `%rhel == 10`, whenever you analyze the
specfile contents.

## 🎯 Original Motivation

The original idea and need arose from [this Copr
issue](https://github.com/fedora-copr/copr/issues/1315), but a similar problem
is being solved in Konflux.

Given a random specfile, along with the `ExcludeArch`, `ExclusiveArch`,
`BuildArch`, etc. statements, the **RPM build system** tries to guess which
architectures it should build for—and consequently, what cloud machines
(architectures) to allocate.  Users often use constructs like:

    %if 0%?fedora > 42
    ExclusiveArch: x86_64 aarch64
    %else
    ExclusiveArch: x86_64
    %endif


Now, which `ExclusiveArch` tag applies to our build?  Consider that the build
system aims to build the package into both "Fedora 42" and "Fedora 43" chroots.
**Both `%if` branches are applicable!**  And what if the build system analyzes
the specfile on a **RHEL 10** machine (building for Fedora)?  In that case,
there's no `%fedora` defined systemwide at all, and `%rhel` exists.

Hence, we need a help with system macro overrides.  Something that allows the
build system to expand macros in a given specfile in a **"speculative" way** for
multiple different "build targets," thus getting the macro expansion closer to
the desired "target" macro expansion.

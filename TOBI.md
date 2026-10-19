# Building QshOni with TOBi (makei)

This fork can be built with IBM i [TOBi](https://ibm.github.io/ibmi-bob/) (the Object Builder, run with `makei`) in addition to the original `build.sh` and `SRCBLDC.CLP` scripts.

## Layout

Sources are grouped in folders by type. File names are unchanged except for two TOBi requirements:

| Folder | Contents | Notes |
|---|---|---|
| `QCMDSRC` | `*.CMD` | TOBi needs command sources in a folder because the source and the `*CMD` object share the `.CMD` extension. |
| `QCLSRC` | `*.CLP`, `*.CLLE` | `CLLE` is built as a `*MODULE` and bound into a program of the same name (ACTGRP `*CALLER`). |
| `QRPGLESRC` | `*.RPGLE`, `*.SQLRPGLE` | `QSHONISRV` builds the service program module. |
| `QPNLSRC` | `QSHONICMDS.PNLGRPSRC` | Panel group with the help text for all commands. Renamed from `.PNLGRP`, which TOBi needs. |
| `QDDSSRC` | `WRKIFSLISD.DSPF` | |
| `QBNDSRC` | `QSHONISRV.BND`, `QSHONISRV.BNDDIR` | `.BND` was `QSHONISRVD.BNDDIR`, the export source. `.BNDDIR` creates the binding directory. |

Each folder has a `Rules.mk`, and the root `Rules.mk` lists the folders. `iproj.json` sets the object library to `QSHONI`.

## Commands and help

Every command is built with its own command processing program and with `HLPPNLGRP(QSHONI/QSHONICMDS)`. `HLPID` defaults to the command name. The panel group is a dependency of every command, so changing it rebuilds them.

## Build order and compile-time needs

- A CL program that calls a QshOni command needs that command to exist when it compiles. `QCLSRC/Rules.mk` lists those commands as dependencies, so TOBi builds them first.
- `QSHQRYSR2R`, `QSHQRYSR3R`, `RUNSQLSRCR` and `RUNSQLSR3R` declare file `TMPQRYSRC`, which the CL programs create in `QTEMP` at run time. The build creates a copy named `QSHONI/TMPQRYSRC` (`RCDLEN(240)`) so they compile. At run time the `QTEMP` file is found first.
- `QSHPORTENC` reads `QTEMP/TCPTMPEND` with `DCLF`, so that file must exist in `QTEMP` when it compiles. Its rule sets `PRECMD`, which TOBi runs in the same job as the compile, to run the `QSHQRYTMP` statement that creates the file. The `QSHQRYTMP` programs are built first.
- TOBi does not pass `DFTACTGRP`, so each RPG program rule adds `DFTACTGRP(*NO)` to match the original build.
- The binding directory source deletes any existing `QSHONISRV` binding directory first, so a rebuild into an existing library works.

## Not built by TOBi


- `QSHONIPR01` to `QSHONIPR04` and `QSHDEMO01R`: sample RPG programs that `/copy qshoni/source,qshonisrvh`, which needs the `QSHONI/SOURCE` source file.
- Data areas, the `QSHMSG` message file and the `QSHONITMP` library that `SRCBLDC.CLP` creates.
- `build.sh` and `SRCBLDC.CLP` expect all sources in one folder. On this branch they no longer find the moved files.

## Build

On the IBM i, with TOBi installed and the project cloned:

```
makei build
```

Build one source:

```
makei compile -f QCMDSRC/QSHEXEC.CMD
```

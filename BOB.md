# Building QshOni with Bob (makei)

This fork can be built with IBM i [Better Object Builder (Bob / TOBi)](https://ibm.github.io/ibmi-bob/) in addition to the original `build.sh` and `SRCBLDC.CLP` scripts.

## Layout

Sources are grouped in folders by type. File names are unchanged except for two Bob requirements:

| Folder | Contents | Notes |
|---|---|---|
| `QCMDSRC` | `*.CMD` | Bob needs command sources in a folder because the source and the `*CMD` object share the `.CMD` extension. |
| `QCLSRC` | `*.CLP`, `*.CLLE` | `CLLE` is built as a `*MODULE` and bound into a program of the same name (ACTGRP `*CALLER`). |
| `QRPGLESRC` | `*.RPGLE`, `*.SQLRPGLE` | `QSHONISRV` builds the service program module. |
| `QPNLSRC` | `QSHONICMDS.PNLGRPSRC` | Panel group with the help text for all commands. Renamed from `.PNLGRP`, which Bob needs. |
| `QDDSSRC` | `WRKIFSLISD.DSPF` | |
| `QBNDSRC` | `QSHONISRV.BND`, `QSHONISRV.BNDDIR` | `.BND` was `QSHONISRVD.BNDDIR`, the export source. `.BNDDIR` creates the binding directory. |

Each folder has a `Rules.mk`, and the root `Rules.mk` lists the folders. `iproj.json` sets the object library to `QSHONI`.

## Commands and help

Every command is built with its own command processing program and with `HLPPNLGRP(QSHONI/QSHONICMDS)`. `HLPID` defaults to the command name. The panel group is a dependency of every command, so changing it rebuilds them.

## Not built by Bob

- `QSHONIPR01` to `QSHONIPR04` and `QSHDEMO01R`: sample RPG programs that `/copy qshoni/source,qshonisrvh`, which needs the `QSHONI/SOURCE` source file.
- Data areas, the `QSHMSG` message file and the `QSHONITMP` library that `SRCBLDC.CLP` creates.
- `build.sh` and `SRCBLDC.CLP` expect all sources in one folder. On this branch they no longer find the moved files.

## Build

On the IBM i, with Bob installed and the project cloned:

```
makei build
```

Build one source:

```
makei compile -f QCMDSRC/QSHEXEC.CMD
```

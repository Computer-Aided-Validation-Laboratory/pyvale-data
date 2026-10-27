# pyvale-data

Data used for the pyvale examples and regression tests.

The files used by pyvale are under `Data/`. The `registry.txt` file
contains SHA256 hashes for those files and is used by pyvale's Pooch data
accessors.

For offline development, point pyvale at this checkout:

```bash
export PYVALE_DATA=/path/to/pyvale-data/Data
```

## Adding data

Place new files inside `Data/` using the path that pyvale should use to refer
to them. For example, a new simulation output belongs in
`Data/simulation/exodus/`, while DIC images belong in the appropriate
`Data/dic/` subdirectory.

After adding or replacing files, regenerate the registry from the repository
root:

```bash
find Data -type f ! -path '*/__pycache__/*' -print0 \
  | sort -z \
  | xargs -0 sha256sum \
  | awk '{path=$2; sub(/^Data\\//, "", path); print path " sha256:" $1}' \
  | sort > registry.txt
```

The registry will also need to be updated in the pyvale repo. Copy the updated registry to
`pyvale/src/pyvale/data/registry.txt`, and update the corresponding pyvale data
accessor if necessary. Registry paths are relative to `Data/`, so they must not include the `Data/` prefix.

Commit the data file and registry changes together. This ensures that every
downloaded file has a matching SHA256 entry when the new pyvale version is
released.

### Archiving large collections

Large multi-file collections can be stored as a ZIP archive instead of as
individual files. For example, the DIC calibration images are stored as
`Data/dic/calibration.zip` and are extracted on demand by pyvale.

Create an archive with the collection contents at its root:

```bash
(cd Data/dic/calibration && zip -r -9 ../calibration.zip .)
```

Add the archive to the registry, remove the uncompressed directory from the
repository, and update the pyvale accessor to use `pooch.Unzip`. Keep the
archive name and extraction path stable because they form part of the cache
layout.

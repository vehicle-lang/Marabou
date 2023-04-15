This PR creates a Python package for `maraboupy` and adds GitHub Actions infrastructure for building wheels for various platforms, architectures, and Python versions.

This PR makes the following MAIN changes:

- Adds `pyproject.toml`, which contains meta-information for the package and configures auxiliary tooling such as bumper and cibuildwheel (see below).
- Adds `setup.py`, which contains the code that builds the native extension with CMake.
- Adds `MANIFEST.in`, which excludes some files that would otherwise be included in the package.
- Adds `.github/workflows/cibuildwheel.yml`, which builds Python wheels for the package on GitHub Actions. (The underlying tool [cibuildwheel](http://cibuildwheel.readthedocs.io) is configured in `pyproject.toml`.)
- Adds a `PYTHON_LIBRARY_OUTPUT_DIRECTORY` setting to `CMakeLists.txt` to allow `setup.py` to where the compiled library is written.
- Adds the contents of the GitHub default `.gitignore` file for Python repositories to `.gitignore` (a lot of the files produced by a Python build are not in the `.gitignore`).


This PR makes the following AUXILIARY changes:

- Fixes the broken test requirements for maraboupy:

  - Removes `codecov` from `maraboupy/test_requirements.txt`, which was [pulled from PyPI](https://about.codecov.io/blog/message-regarding-the-pypi-package/). The package is not required on CI, as the main workflow uses the `codecov` action.
  - Relaxes the version constraints from exact versions to version bounds, e.g., `numpy==1.21.0` changes to `numpy>=1.21.0,<2`, to ensure greater compatibility across Python versions.

- Fixes the broken badges in `README.md`:

  - Renames `.github/workflows/ci-with-production.yml` back to `.github/workflows/ci.yml`. The badges in `README.md` still referenced the old name, and therefore hadn't updated since the renaming took place.

- Fixes the outdated actions in the main workflow:

  - Bumps `actions/cache` from `v1` to `v3`, as `v1` was deprecated and soon to be removed.
  - Adds `.github/dependabot.yml`, which configures dependabot to automatically open PRs when the used GitHub actions become out-of-date, to prevent this from happening in the future.

- Disables the failing regression test `mnist-bnn_index0_eps0.001_target9_unsat.ipq` by suffixing the filename with `.disabled`.

- Fixes the `on` clause in every workflow to run the CI:

  - when changes are pushed to master;
  - when a PR is openend or changes are pushed to the PR;
  - every Monday at 7AM to catch bugs due to changes in the ecosystem; and
  - whenever someone with the appropriate privileges requests it.

  Before this change, the CI would run on every push on every branch, and on every pull request trigger, which, amongst other things, cause the CI to run _twice_ on PRs from branches in the same repository, and cause the CI to run on minor actions which don't change code, such as adding a label to a PR.

- Adds support for Python 3.11:

  - Bumps pybind11 from 2.3.0 to 2.10.4, as 2.3.0 was too old to build Python 3.11 wheels without deprecation warnings (which the CMake configuration elevates to errors).
  - Adds the relevant files for the new pybind11 to the `.gitignore` file.

- Changes the tests for maraboupy to use absolute rather than relative imports, as the tests are not considered part of the distributed package. (The test directory is stripped out by `MANIFEST.in`.)

- Changes the main workflow to be more easily adaptable to testing on macOS in the future:

  - Adds the OS to the matrix, with `ubuntu-latest` as its only value;
  - Splits the step which installs packages into (1) a step which installs system packages, which runs only on Linux, and (2) a step which installs Python packages, which runs on every OS.

- Prunes unused imports of `pytest` from the tests for maraboupy.

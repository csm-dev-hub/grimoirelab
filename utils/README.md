# Utils

Utils, general to GrimoireLab.

## build_grimoirelab

Build packages (sdist, wheel) for all modules in GrimoireLab.
This program can build the packages corresponding to a coordinated release
of GrimoireLab, or to the current master/HEAD of all repos.
The build can use virtual environments for building everything,
can be installed for testing in a virtual environment too,
and can be checked for some invariants (such as modification of the
version file).
A set of modules to install can be selected, if we don't want to build
all of them.

Currently, this script can produce all packages for the repos that
already support Python packages,
and install them in a virtual environment where they could be tested.

To avoid problems with the Python installation in the host
where the packages are produced,
a virtual environment with installed dependencies for building is produced,
and packages are built in it.

The script accepts options to build, install, or check,
to select only a certain set of modules,
to define directories for building and installing virtual environments,
for the temporary repositories, for where to produce the Python packages,
and to select a coordinated release file.

The output (stdout) shows a summary of the process, including builds
and checks that failed. In general, any command that fails causes the
script to finish.

### Examples

* Build all modules in a coordinated release of GrimoireLab
(the one corresponding to `../releases/elasticgirl.16`),
producing the corresponding wheels and sdists in `/tmp/dist`,
cloning all git repos in `/tmp/repos`,
using for building a virtual environment in `/tmp/venv`,
installing all the modules in the virtual environment `/tmp/ivenv`
afterwards, including dependencies,
and running some checks on them.
Write log file in `/tmp/log`, using log level `debug`.

```bash
$ build_grimoirelab -l debug --logfile /tmp/log --build --install --check \
  --relfile ../releases/elasticgirl.16  --dist /tmp/dist \
  --reposdir /tmp/repos --venv /tmp/venv --install_venv /tmp/ivenv
```

The stdout shows how the process went, including builds or tests that
failed:

```
Building...
Built module perceval-opnfv: OK
Built module perceval-mozilla: OK
Built module perceval: OK
Built module grimoirelab-toolkit: OK
Built module grimoire-elk: OK
Built module sortinghat: OK
Built module grimoire-kidash: OK
Installing...
Installed modules: perceval-opnfv,perceval-mozilla,perceval,grimoirelab-toolkit,grimoire-elk,sortinghat,grimoire-kidash
Checking...
Checked version for perceval-opnfv: Fail
Checked version for perceval-mozilla: OK
Checked version for perceval: Fail
Checked version for grimoirelab-toolkit: Fail
Checked version for grimoire-elk: Fail
Checked version for sortinghat: OK
Installed modules (all):
Venv for building in /tmp/venv
Repos for source code in /tmp/repos
Distribution packages in /tmp/dist
Installed modules in /tmp/ivenv
```

* Build all modules in a coordinated release of GrimoireLab
(the one corresponding to `../releases/elasticgirl.16`),
letting the script decide which directories to use, producing
minimum messages, and no log file. This can be used as a "minimal
check" for a release.

```bash
$ build_grimoirelab --build --install --check \
  --relfile ../releases/elasticgirl.16
```

* Build all modules in a coordinated release of GrimoireLab
(the one corresponding to `../releases/elasticgirl.16`),
producing the corresponding wheels and sdists in `/tmp/dist`,
installing them in the virtual environment `/tmp/ivenv`
afterwards, including dependencies,
and running some checks on them.
Write log file in `/tmp/log`, using log level `debug`.

```bash
$ build_grimoirelab -l debug --logfile /tmp/log --build --install --check \
  --relfile ../releases/elasticgirl.16  --dist /tmp/dist \
  --install_venv /tmp/ivenv
```

* Build the `grimoire-kidash` module, as is in the `master/HEAD` commit
of its repository,
producing the corresponding wheels and sdists in `/tmp/dist`.

```bash
$ build_grimoirelab --build --relfile ../releases/elasticgirl.16  \
  --dist /tmp/dist  --modules grimoire-kidash
```

* Install modules available in `/tmp/dist` as wheels and sdists,
in the virtual environment `/tmp/ivenv`, including dependencies.

```bash
$ build_grimoirelab --install --dist /tmp/dist --install_venv /tmp/ivenv \
  --modules grimoire-kidash
```

* Build (creating packages in `/tmp/dist`, install, check, and run tests,
overriding the default list of repos with those in `repos.json`,
and using the commits specified in `release`

```
$ build_grimoirelab --build --install --check --test \
  --dist /tmp/dist --reposfile repos.json --relfile release
```

The file `repos.json` can be as follows:

```
{
  "perceval": [
    {
      "name": "perceval",
      "dir": "",
      "repo_url": "/src/chaoss/grimoirelab-perceval",
      "version_file": "perceval/_version.py",
      "check_run": "perceval --help",
      "test": true
    }
  ],
  "perceval-mozilla": [
    {
      "name": "perceval-mozilla",
      "dir": "",
      "repo_url": "/src/chaoss/grimoirelab-perceval-mozilla",
      "version_file": "setup.py",
      "check_run": "perceval kitsune --help"
    }
  ]
}
```

* Run tests for all repos with tests activated (`test` is true),
using for each repo the checkout in `18.06-02`,
and prebuilt packages in the `dir` directory for Installing
dependencies for running tests:

```
./build_grimoirelab --test --relfile 18.06-02 \
  --distdir dist --fail
```
* In some cases, running tests require of using some specific files,
such as customization files, that need to be copied to the repository
where tests will be run. For that, we have the `--confdir` argument.
`--confdir` specifies a directory where it is expected to have one
subdirectory per module needing files to be copied,
with files in the same relative path to where they should be copied:

```
./build_grimoirelab --test --relfile 18.06-02 \
  --confdir conf --distdir dist --fail
```

* We can also obtain a list of all the top level dependencies
that we need to install a certain release of GrimoireLab (that is,
the list of GrimoireLab modules with their version numbers),
suitable for being used as a `requirements.txt` file:

```
./build_grimoirelab --relfile ../releases/18.07-05 \
  --dependencies --depfile requirements.txt
```

For complete list of arguments, run:

```bash
$ build_grimoirelab --help
```


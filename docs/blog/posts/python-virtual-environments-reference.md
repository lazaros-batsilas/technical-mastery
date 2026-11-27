# Python Virtual Environments: Troubleshooting and Command Reference

This companion guide expands on the practical diagnostics and commands introduced in [Python Virtual Environments: From Setup to Understanding](python-virtual-environments.md).

## 1. Troubleshooting and Diagnostics

### 1.1 Interpreter and Environment Diagnostics

Start by identifying the interpreter actually running the code, rather than relying on the terminal prompt or the interpreter selected in VS Code.

```bash
which python3
python3 -c "import sys; print(sys.executable, sys.prefix, sys.base_prefix)"
echo "$VIRTUAL_ENV"
echo "$PATH"
```

For a standard `venv` environment, the following Python expression returns `True` when the running interpreter belongs to a virtual environment:

```python
import sys
print(sys.prefix != sys.base_prefix)
```

The runtime prefixes identify the Python environment, whereas `VIRTUAL_ENV` indicates shell activation. See [Section 2.7](#27-comprehensive-python-diagnostic-script) for a comprehensive diagnostic script and [Appendix A](python-virtual-environments-appendix.md) for the expected values in different execution scenarios.

### 1.2 VS Code and Terminal Interpreter Mismatches

Common symptoms include:

- A script runs successfully in VS Code but produces `ModuleNotFoundError` in the terminal.
- A package is installed successfully in the terminal but cannot be imported when running a script through VS Code.
- A newly opened integrated terminal uses system Python despite the correct interpreter being selected in VS Code.

These problems often arise because interpreter selection and terminal activation are separate mechanisms.

**Diagnosis:** Check `Python: Select Interpreter` in the Command Palette and compare the runtime diagnostic script in [Section 2.7](#27-comprehensive-python-diagnostic-script) when executed through VS Code and the terminal. Inspect the terminal's package installation location and compare package inventories:

```bash
which python3
python3 -m pip --version
python3 -m pip show numpy
.venv/bin/python -m pip show numpy
```

NumPy is used here as an illustrative package.

**Resolution:** Select `.venv/bin/python` in VS Code. Check the extension's automatic terminal activation settings, which may vary by extension version. Close the existing terminal and open a new one. If activation still fails, activate manually from the project root. Below `verify_environment.py` is defined in [Section 2.7](#27-comprehensive-python-diagnostic-script)

```bash
source .venv/bin/activate
python verify_environment.py
```

Alternatively, invoke the intended interpreter directly:

```bash
.venv/bin/python verify_environment.py
```

Only if the required package is missing from the intended environment, install it there:

```bash
.venv/bin/python -m pip install numpy
```

**Verification:** Confirm the interpreter using `sys.executable`, check the import and rerun the original script through both VS Code and the terminal.

```bash
.venv/bin/python -c "import numpy; print(numpy.__version__)"
```

If the import still fails, inspect the traceback, the import name and any local files shadowing the package.

### 1.3 Shell Activation and Environment Persistence

Two common sources of confusion are a prompt displaying `(.venv)` while Python runs outside the environment, and the apparent disappearance of the environment after closing a terminal.

In the first case, an explicitly invoked system interpreter or overridden command resolution may bypass the activated environment. Compare these commands in an activated terminal (alternatively you can use `verify_environment.py` in [Section 2.7](#27-comprehensive-python-diagnostic-script)):

[Appendix A](python-virtual-environments-appendix.md) contains the expected values for different scenaria.

```bash
python3 -c "import sys; print(sys.executable, sys.prefix, sys.base_prefix)"
/usr/bin/python3 -c "import sys; print(sys.executable, sys.prefix, sys.base_prefix)"
```

Note that closing the terminal discards the shell's activation state, not the virtual environment or its installed packages. Check that the directory exists and reactivate it:

```bash
ls -ld .venv
source .venv/bin/activate
python -m pip list
```

There is no need to recreate an existing environment unless it is missing, damaged or incompatible with the current system.

### 1.4 Dependency Installation and Version Conflicts

| Symptom | Possible cause | Corrective action |
|---|---|---|
| `pip install requirements.txt` fails | Missing `-r` option | Use `python -m pip install -r requirements.txt`. |
| Developers install different versions | Flexible version constraints | Review constraints; use exact pins where required. |
| Installation reports conflicts or an application fails after an upgrade | Incompatible dependencies or application-level incompatibility | Inspect installed versions and declared requirements before changing packages. |

Run the following commands in the intended environment:

```bash
python -m pip list
python -m pip freeze
python -m pip check
python -m pip show package-name
```

Replace `package-name` with the relevant package. A constraint such as `example-package>=2.4.0` permits both 2.4.0 and 3.0.0; `example-package==2.4.0` requires exactly 2.4.0. A single equals sign is not valid exact-version syntax.

After reviewing the requirements file, restore dependencies using:

```bash
python -m pip install -r requirements.txt
python -m pip check
```

If installation fails, inspect the error for incompatible constraints, unsupported Python versions or unavailable distributions. If a known-good requirements file exists, consider recreating the environment in a fresh directory and testing it before replacing the old one. Avoid blindly upgrading all packages.

A successful `pip check` confirms declared dependency consistency, not application correctness. Run the project's tests as well.

### 1.5 Git Tracking and Virtual Environment Exclusion

**Symptom:** Files inside `.venv` appear in Git even though the directory is listed in `.gitignore`.

**Investigation:**

```bash
git check-ignore -v .venv/
git ls-files -- .venv/
git status --short --ignored
```

**Likely cause:** Files were tracked before the ignore rule was added. If no ignore rule is reported, inspect `.gitignore` and the path. For tracked paths, `git check-ignore` normally suppresses output; `git check-ignore -v --no-index .venv/` can help inspect the rule independently of tracking.

**Solution:** Ensure `.venv/` is in `.gitignore`. Only if the directory is already tracked, remove its files from the index while retaining the local copies:

```bash
git rm -r --cached .venv/
git diff --cached --stat
```

Review the staged changes and commit them together with the ignore rule if it was changed. This does not remove the files from earlier commits.

**Verification:** `git ls-files -- .venv/` should produce no output, and `git status --short --ignored` should report `!! .venv/` (alongside any other repository changes).

## 2. Comprehensive Command Reference

The following reference assumes Linux with a Bash-compatible shell and commands executed from the project root. Use `python3` before activation if `python` is unavailable. Package and script names are illustrative placeholders. Commands that install, remove or upgrade packages should be run only in the intended environment.

### 2.1 Environment Lifecycle

| Command | Purpose |
|---|---|
| `pwd` | Display the current working directory |
| `python3 -m venv --help` | Display virtual environment creation options |
| `python3 -m venv .venv` | Create a virtual environment |
| `source .venv/bin/activate` | Activate it in the current shell |
| `deactivate` | Restore the previous shell configuration |
| `ls -la .venv` | Inspect the environment directory |
| `cat .venv/pyvenv.cfg` | Inspect the environment configuration |
| `.venv/bin/python --version` | Check the environment's Python version without activation |
| `.venv/bin/python script.py` | Execute a script without activation |
| `/usr/bin/python3 script.py` | Explicitly execute a script using system Python on this installation |

### 2.2 Interpreter and Shell Diagnostics

| Command | Purpose |
|---|---|
| `python --version` | Display the selected Python version |
| `python3 --version` | Display the Python 3 version |
| `which python` | Locate the Python executable on PATH |
| `which python3` | Locate the Python 3 executable on PATH |
| `which pip` | Locate the pip executable on PATH |
| `command -v python3` | Inspect shell command resolution |
| `type -a python3` | Show matching commands, aliases or functions |
| `echo "$VIRTUAL_ENV"` | Inspect the shell activation variable |
| `echo "$PATH"` | Inspect the executable search path |
| `python -c "import sys; print(sys.executable)"` | Display the running Python executable |
| `python -c "import sys; print(sys.prefix, sys.base_prefix)"` | Compare runtime prefixes |
| `python -c "import sys; print(sys.prefix != sys.base_prefix)"` | Detect execution inside a standard venv |
| `python -m site` | Inspect Python's module search paths and site configuration |

### 2.3 Package Management

| Command | Purpose |
|---|---|
| `python -m pip --version` | Display pip version and installation location |
| `python -m pip list` | List installed packages |
| `python -m pip list --outdated` | Identify packages with newer available versions |
| `python -m pip show package-name` | Display package metadata, location and dependencies |
| `python -m pip install package-name` | Install a package |
| `python -m pip install mkdocs-material` | Install MkDocs Material |
| `python -m pip install "package-name==2.5.0"` | Install an exact version |
| `python -m pip install --upgrade package-name` | Upgrade a package, subject to constraints |
| `python -m pip uninstall package-name` | Uninstall a package |
| `.venv/bin/python -m pip install package-name` | Install directly into the project environment |
| `python -m pip check` | Check for missing or incompatible declared dependencies |
| `python -m pip --help` | Display pip commands and options |
| `python -m pip install --help` | Display installation options |

!!! note "Executing pip"

    Running `python -m pip` invokes pip through the specified Python interpreter. This reduces the risk of installing packages into the wrong environment. It does not itself guarantee that the chosen interpreter is the intended one.

### 2.4 Dependency Export and Restoration

| Command | Purpose |
|---|---|
| `python -m pip freeze` | Display installed packages in requirements format |
| `python -m pip freeze --all` | Include packages normally excluded from freeze output |
| `python -m pip freeze > requirements.txt` | Export installed packages, overwriting the destination file |
| `cat requirements.txt` | Inspect the requirements file |
| `python -m pip install -r requirements.txt` | Install dependencies specified in the file |
| `python -m pip check` | Verify declared dependency consistency |
| `git diff -- requirements.txt` | Review unstaged changes to a tracked requirements file |

`pip freeze` records installed packages rather than identifying only direct dependencies. Review its output before committing it, especially if the environment contains unrelated packages or local paths.

### 2.5 Git Integration and Verification

| Command | Purpose |
|---|---|
| `cat .gitignore` | Inspect repository ignore rules |
| `grep -n 'venv' .gitignore` | Find relevant rules with line numbers |
| `git check-ignore -v .venv/` | Identify the applicable ignore rule |
| `git check-ignore -v --no-index .venv/` | Inspect ignore matching independently of the index |
| `git ls-files -- .venv/` | List tracked files inside the environment |
| `git status` | Inspect repository status |
| `git status --short` | Display compact status |
| `git status --short --ignored` | Include ignored paths in compact status |
| `git rm -r --cached .venv/` | Stage removal of previously tracked environment files while retaining local copies |
| `git add -- .gitignore requirements.txt` | Stage the ignore rules and dependency file, when present |
| `git diff --cached` | Review staged changes before committing |
| `git diff --cached --stat` | Summarise staged changes |

The `--` separator distinguishes options from path arguments. An empty result from `git ls-files -- .venv/` means no environment files are tracked. In short status output, `??` means untracked and `!!` means ignored.

### 2.6 VS Code Commands

| Command or action | Purpose |
|---|---|
| `code .` | Open the current project in VS Code, if its CLI is available |
| `code --list-extensions` | List installed extensions |
| `code --install-extension ms-python.python` | Install the Microsoft Python extension |
| `Ctrl+Shift+P` | Open the Command Palette |
| `Python: Select Interpreter` | Select the project interpreter |
| `Python: Run Python File in Terminal` | Run the current Python file using the Python extension |
| `Terminal: Create New Terminal` | Open a new integrated terminal |

Interpreter selection and terminal activation should be verified separately.

### 2.7 Comprehensive Python Diagnostic Script

Save the following as `verify_environment.py` if a reusable diagnostic script is needed:

```python
import os
import site
import sys

print("Python version:", sys.version)
print("Python executable:", sys.executable)
print("Current prefix:", sys.prefix)
print("Base prefix:", sys.base_prefix)
print("Running inside venv:", sys.prefix != sys.base_prefix)
print("VIRTUAL_ENV:", os.environ.get("VIRTUAL_ENV"))
print("Package directories:", site.getsitepackages())
```

Compare the output when running through VS Code and when executing the following commands from the project root:

```bash
python3 verify_environment.py
.venv/bin/python verify_environment.py
/usr/bin/python3 verify_environment.py
```

Repeat before and after activation. The expected prefix values for the installation used in this article are listed in [Appendix A](python-virtual-environments-appendix.md). The directories returned by `site.getsitepackages()` are candidate package locations and need not all exist.

## Related Reading

- [Main article](python-virtual-environments.md)
- [Appendix A: Interpreter and Prefix Comparison Tables](python-virtual-environments-appendix.md)

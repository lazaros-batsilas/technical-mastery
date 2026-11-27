# Appendix A: Python Interpreter and Prefix Comparison Tables

Companion to [Python Virtual Environments: From Setup to Understanding](python-virtual-environments.md) and the [Troubleshooting and Command Reference](python-virtual-environments-reference.md).

The tables cover eight valid execution scenarios for the commands considered. The exact paths may differ on other systems. The command `python3` is used because `python` is not necessarily available outside a virtual environment on Linux. Assume commands are executed from the project root, with normal shell configuration and no Python path overrides. `PROJECT` represents the absolute project directory.

## Case I. No virtual environment has been created

| Attribute | `python3` | `/usr/bin/python3` |
|---|---|---|
| `sys.executable` | `/usr/bin/python3` | `/usr/bin/python3` |
| `sys.prefix` | `/usr` | `/usr` |
| `sys.base_prefix` | `/usr` | `/usr` |
| `VIRTUAL_ENV` | Not set | Not set |
| Running inside a virtual environment? | No | No |

Executing `.venv/bin/python` is not possible because the virtual environment does not exist.

## Case II. Virtual environment exists and is activated

Activation prepends `PROJECT/.venv/bin` to the shell's `PATH`.

| Attribute | `python3` | `.venv/bin/python` | `/usr/bin/python3` |
|---|---|---|---|
| `sys.executable` | `PROJECT/.venv/bin/python3` | `PROJECT/.venv/bin/python` | `/usr/bin/python3` |
| `sys.prefix` | `PROJECT/.venv` | `PROJECT/.venv` | `/usr` |
| `sys.base_prefix` | `/usr` | `/usr` | `/usr` |
| `VIRTUAL_ENV` | `PROJECT/.venv` | `PROJECT/.venv` | `PROJECT/.venv` |
| Running inside a virtual environment? | Yes | Yes | No |

**Important:** Explicitly executing `/usr/bin/python3` bypasses the virtual environment's interpreter even when the shell is activated.

## Case III. Virtual environment exists but is deactivated

Deactivation restores the previous `PATH` without deleting the virtual environment.

| Attribute | `python3` | `.venv/bin/python` | `/usr/bin/python3` |
|---|---|---|---|
| `sys.executable` | `/usr/bin/python3` | `PROJECT/.venv/bin/python` | `/usr/bin/python3` |
| `sys.prefix` | `/usr` | `PROJECT/.venv` | `/usr` |
| `sys.base_prefix` | `/usr` | `/usr` | `/usr` |
| `VIRTUAL_ENV` | Not set | Not set | Not set |
| Running inside a virtual environment? | No | Yes | No |

**The decisive factor is which Python interpreter is executed, not whether the virtual environment is activated.**

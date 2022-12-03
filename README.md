# python-mono

Examples of problems that come when trying to use explicit relative imports in Python.

## Scripts Can't Import Relative

Taken from https://stackoverflow.com/questions/14132789/relative-imports-for-the-billionth-time, relative imports
are only relative inside a package.

### Example:

Run mono1/testa.sh from the mono1 directory.

```bash
mono1$ source testa.sh
python3 apps/calc/calculate.py
Traceback (most recent call last):
  File "/home/xentropy/Source/mono-py/mono1/apps/calc/calculate.py", line 1, in <module>
    from .helper import get_help
ImportError: attempted relative import with no known parent package
```

### Solution:

Run as module:

```bash
mono1$ source testb.sh
python3 -m apps.calc.calculate
Please, help me.
120
```

## CWD Is Not A Package

Python doesn't consider the current working directory a package.
https://stackoverflow.com/questions/30669474/beyond-top-level-package-error-in-relative-import

### Example:

From the mono1 directory

```bash
mono1$ source testc.sh
python3 -m apps.calc.compute
Traceback (most recent call last):
  File "/usr/lib/python3.10/runpy.py", line 196, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/usr/lib/python3.10/runpy.py", line 86, in _run_code
    exec(code, run_globals)
  File "/home/xentropy/Source/mono-py/mono1/apps/calc/compute.py", line 4, in <module>
    from ...lib.math.misc import factorial
ImportError: attempted relative import beyond top-level package
```

The line in question causing the problem is:

```python
from ...lib.math.misc import factorial
```

The problem is `...` from `mono1.apps.calc` is `mono1`, and you are running the command from the `mono1` directory.
Python does not consider the current working directory a package.

### Solution:

Run the module from one directory higher.

```bash
mono-py$ source testd.sh
python3 -m mono1.apps.calc.compute
Please, help me.
120
```

## Other things to consider

- How do you deploy a project like this? With all of the relative imports, the folder structure needs to stay intact.
  - You could copy _only_ the folders necessary, or you could copy the entire python mono, or you could use some build tool
- Is it worth it to clutter the entire mono-repo with `__init__.py` files instead of just creating a python folder?
- How are dependencies handled per app? Is there one virtual environment in the root of the mono, or many all over?
- What directory you run things from matters. You need to be 1 directory higher than the package directory (see "CWD Is Not A Package")
  - it sort of makes sense to have one package.json for launching all of the apps. Poetry can also accomplish this task

## Opinions

I'm not sure it's worth the added overhead and complexity of including Python in a polyglot mono-repo. Due to limitations
in how Python handles explicity relative imports, and the lack of built in support for mono-repos in tools like Poetry,
it's kind of a mess to maintain. Open to ideas/suggestions.

If building a python mono repo, definitely need to consider the limitations when building the folder structure.
I'd recommend somthing like this:

- /
  - package.json
  - pyproject.toml
  - mono/
    - apps/
      - app1/
      - app2/
    - lib/
      - lib1/
      - lib2/

With `__init__.py` files in every folder except /.
All modules with / as the current working directory.
Deploy in the exact same folder structure but do not copy files that the module doesn't use.
Ie: app1 might deploy with the whole structure but without app2.

## Final remarks

This is dumb, plz fix in Python 4.

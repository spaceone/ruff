# project

An example multi-package Python project used to test setting resolution and other complex
behaviors.

## Expected behavior

Running from the repo root should pick up and enforce the appropriate settings for each package:

```
∴ cargo run resources/test/project/
resources/test/project/examples/.dotfiles/script.py:1:1: I001 Import block is un-sorted or un-formatted
resources/test/project/examples/.dotfiles/script.py:1:8: F401 `numpy` imported but unused
resources/test/project/examples/.dotfiles/script.py:2:17: F401 `app.app_file` imported but unused
resources/test/project/examples/docs/docs/file.py:1:1: I001 Import block is un-sorted or un-formatted
resources/test/project/examples/docs/docs/file.py:8:5: F841 Local variable `x` is assigned to but never used
resources/test/project/project/file.py:1:8: F401 `os` imported but unused
resources/test/project/project/import_file.py:1:1: I001 Import block is un-sorted or un-formatted
Found 7 error(s).
7 potentially fixable with the --fix option.
```

Running from the project directory itself should exhibit the same behavior:

```
∴ (cd resources/test/project/ && cargo run .)
examples/.dotfiles/script.py:1:1: I001 Import block is un-sorted or un-formatted
examples/.dotfiles/script.py:1:8: F401 `numpy` imported but unused
examples/.dotfiles/script.py:2:17: F401 `app.app_file` imported but unused
examples/docs/docs/file.py:1:1: I001 Import block is un-sorted or un-formatted
examples/docs/docs/file.py:8:5: F841 Local variable `x` is assigned to but never used
project/file.py:1:8: F401 `os` imported but unused
project/import_file.py:1:1: I001 Import block is un-sorted or un-formatted
Found 7 error(s).
7 potentially fixable with the --fix option.
```

Running from the sub-package directory should exhibit the same behavior, but omit the top-level
files:

```
∴ (cd resources/test/project/examples/docs && cargo run .)
docs/file.py:1:1: I001 Import block is un-sorted or un-formatted
docs/file.py:8:5: F841 Local variable `x` is assigned to but never used
Found 2 error(s).
2 potentially fixable with the --fix option.
```

`--config` should force Ruff to use the specified `pyproject.toml` for all files, and resolve
file paths from the current working directory:

```
∴ (cargo run -- --config=resources/test/project/pyproject.toml resources/test/project/)
resources/test/project/examples/.dotfiles/script.py:1:8: F401 `numpy` imported but unused
resources/test/project/examples/.dotfiles/script.py:2:17: F401 `app.app_file` imported but unused
resources/test/project/examples/docs/docs/concepts/file.py:1:8: F401 `os` imported but unused
resources/test/project/examples/docs/docs/file.py:1:1: I001 Import block is un-sorted or un-formatted
resources/test/project/examples/docs/docs/file.py:1:8: F401 `os` imported but unused
resources/test/project/examples/docs/docs/file.py:3:8: F401 `numpy` imported but unused
resources/test/project/examples/docs/docs/file.py:4:27: F401 `docs.concepts.file` imported but unused
resources/test/project/examples/excluded/script.py:1:8: F401 `os` imported but unused
resources/test/project/project/file.py:1:8: F401 `os` imported but unused
Found 9 error(s).
9 potentially fixable with the --fix option.
```

Running from a parent directory should "ignore" the `exclude` (hence, `concepts/file.py` gets
included in the output):

```
∴ (cd resources/test/project/examples && cargo run -- --config=docs/ruff.toml .)
docs/docs/concepts/file.py:5:5: F841 Local variable `x` is assigned to but never used
docs/docs/file.py:1:1: I001 Import block is un-sorted or un-formatted
docs/docs/file.py:8:5: F841 Local variable `x` is assigned to but never used
excluded/script.py:5:5: F841 Local variable `x` is assigned to but never used
Found 4 error(s).
4 potentially fixable with the --fix option.
```

Passing an excluded directory directly should report errors in the contained files:

```
∴ cargo run resources/test/project/examples/excluded/
resources/test/project/examples/excluded/script.py:1:8: F401 `os` imported but unused
Found 1 error(s).
1 potentially fixable with the --fix option.
```

Unless we `--force-exclude`:

```
∴ cargo run resources/test/project/examples/excluded/ --force-exclude
warning: No Python files found under the given path(s)
∴ cargo run resources/test/project/examples/excluded/script.py --force-exclude
warning: No Python files found under the given path(s)
```

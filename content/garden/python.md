+++
title = "python"
author = ["Nilay Kumar"]
date = 2025-02-26T00:00:00-05:00
lastmod = 2025-03-01T18:45:29-05:00
tags = ["code", "python"]
draft = false
progress = "new"
+++

## package management {#package-management}


### uv {#uv}

I'm using [uv](https://docs.astral.sh/uv/) to manage python versions and per-project virtual environments.


#### uvx/uv tool {#uvx-uv-tool}

One interesting feature of `uv` is the ability to run tools without installing
them, e.g. `uvx ruff` (which is just an alias for `uv tool run ruff`) to run the
code format without requiring it in the project. This will install the tool into
a temporary, isolated environment and then run it. Note that this workflow
therefore requires an internet connection, and does not work offline. Instead we
can run `uv tool install ruff` to remove the need to re-install the tool every
time we run it. The tool should then be visible in the `PATH`, but its modules
will not be available for use in the project.


#### scripts (uv run) {#scripts--uv-run}

We can run one-off scripts (outside of a project) with `uv run` as long as we
specify the dependencies explicitly. For example,

```sh
uv run --with "requests<3, pandas[pyarrow]" script.py
```

will temporarily install the specified dependencies and run the script in that
temporary environment. This is convenient for those one-off scripts that you
don't necessarily want to make a new directory, etc. etc. As [PEP 723](https://peps.python.org/pep-0723/) notes:

> Python is routinely used as a scripting language, with Python scripts as a
> (better) alternative to shell scripts, batch files, etc. When Python code is
> structured as a script, it is usually stored as a single file and does not
> expect the availability of any other local code that may be used for imports. As
> such, it is possible to share with others over arbitrary text-based means such
> as email, a URL to the script, or even a chat window. Code that is structured
> like this may live as a single file forever, never becoming a full-fledged
> project with its own directory and pyproject.toml file.

The ergonomics of running scripts with a lengthy list of requirements is not
great, but `uv` is able to take advantage of the metadata specification
proposed by PEP 723 to use requirements specified in the script itself. This
looks something like

```python
# /// script
# requires-python = ">=3.13"
# dependencies = [
#     "pandas[pyarrow]",
#     "requests<3",
# ]
# ///
```

You don't need to write this by hand -- `uv` can do it for you:

```sh
uv add --script script.py "requests<3" "pandas[pyarrow]"
```


## notebooks {#notebooks}


### marimo {#marimo}


## web {#web}


### fastapi {#fastapi}


### fasthtml {#fasthtml}


### webasm {#webasm}

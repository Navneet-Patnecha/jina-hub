# Jina Hub

Jina Hub is an open-registry for hosting Jina executors via container images. It enables users to ship and exchange reusable component across various Jina search applications.

From Jina 0.4.10, Jina Hub is referred as a Git Submodule in [`jina-ai/jina`](https://github.com/jina-ai/jina).

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->



<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## Use Hub Image in Flow API

```python
# my_image_tag is the the built image name

from jina import Flow
with Flow().add(uses='docker://' + my_image_tag):
    pass
```

## Create a new Hub Pod/App

### Prerequisites

- Install [Docker](https://docs.docker.com/get-docker/).
- `pip install "jina[hub]"`

### Create a New Executor
```bash
jina hub new --type pod
```

It will start a wizard in CLI to guide you create your first executor. The resulted file structure should look like the following:

```text
- MyAwesomeExecutor/
    |
    |- Dockerfile
    |- manifest.yml
    |- README.md
    |- requirements.txt
    |- __init__.py
    |- config.yml
    |- tests/
        |- test_MyAwesomeExecutor.py
        |- __init__.py
```

### Use `py_modules` to Import Multiple Files

By default, `jina hub new` creates a Python module structure and guides you to write `MyAwesomeExecutor` class into `__init__.py`. If you have some other files that need to be imported for `MyAwesomeExecutor`, say `helper.py`, you can change [`metas.pymodules`](https://docs.jina.ai/api/jina.executors.metas.html?highlight=py_modules#confval-py_modules) in `config.yml` to import those files. Note, you have to write the dependency in reverse order. That is, if `__init__.py` depends on `A.py`, which again depends on `B.py`, then you need to write:

```yaml
!MyAwesomeExecutor
with:
  ...
metas:
  py_modules:
    - B.py
    - A.py
    - __init.py
```

#### Legacy Hub Pod Structure

This is also a valid structure, it works but not recommended:


```text
- MyAwesomeExecutor/
    |
    |- Dockerfile
    |- manifest.yml
    |- README.md
    |- requirements.txt
    |- MyAwesomeExecutor.py
    |- helper.py
    |- config.yml
        |- metas.py_modules
            |- helper.py
            |- MyAwesomeExecutor.py
```

Please note that:
    - Here `MyAwesomeExecutor` directory is not a python module, as it lacks `__init__.py` under the root;
    - To import ``foo.py``, you must to use ``from jinahub.foo import bar``, where ``jinahub`` is the common namespace for all external modules;
    - In `config.yml:metas.py_modules`, ``helper.py`` needs to be put before `MyAwesomeExecutor.py` in YAML ``py_modules``.



### Test an Pod/App Locally

```bash
jina hub build /MyAwesomeExecutor/
```

More Hub CLI usage can be found via `jina hub build --help`

## Work with Your Own Repository

1. Create a new repository
2. `pip install "jina[hub]" && jina hub new --type pod`
3. 
```
git checkout -b feat-new-executor
git add .
git commit -m "feat: add new executor"
git push
```
4. Add the [Hub Updater](https://github.com/jina-ai/action-hub-updater) and [Hub Builder](https://github.com/jina-ai/action-hub-builder) Github Actions to the Github workflow of your hub-type repo.
5. Make a Pull Request on `feat-new-executor -> master`


## References

### Schema of `manifest.yml`

`manifest.yml` must exist if you want to publish your Pod image to Jina Hub.

`manifest.yml` annotates your image so that it can be managed by Jina Hub. To get better appealing on Jina Hub, you should carefully set `manifest.yml` to the correct values.

| Key | Description | Default |
| --- | --- | --- |
| `manifest_version` | The version of the manifest protocol | `1` |
| `type` | The type of the image | Required |
| `kind` | The kind of the executor | Required |
| `name` | Human-readable title of the image (must match with `^[a-zA-Z_$][a-zA-Z_\s\-$0-9]{3,30}$`) | Required |
| `description` | Human-readable description of the software packaged in the image | Required |
| `author` | Contact details of the people or organization responsible for the image (string) | `Jina AI Dev-Team (dev-team@jina.ai)` |
| `url` | URL to find more information on the image (string) | `https://jina.ai` |
| `documentation` | URL to get documentation on the image (string) | `https://docs.jina.ai` |
| `version` | Version of the image, it should be [Semantic versioning-compatible](http://semver.org/) | `0.0.0` |
| `vendor` | The name of the distributing entity, organization or individual (string) | `Jina AI Limited` |
| `license` | License under which contained software is distributed, it should be [in this list](legacy/builder/osi-approved.yml) | `apache-2.0` |
| `avatar` | A picture that personalizes and distinguishes your image | None |
| `platform` | A list of CPU architectures that your image built on, each item should be [in this list](legacy/builder/platforms.yml) | `[linux/amd64]` |
| `keywords` | A list of strings help user to filter and locate your package  | None | 


## Contributing

We welcome all kinds of contributions from the open-source community, individuals and partners. Without your active involvement, Jina won't be successful.

Please first read [the contributing guidelines](https://github.com/jina-ai/jina/blob/master/CONTRIBUTING.md) before the submission. 

## License

Copyright (c) 2020 Jina AI Limited. All rights reserved.

Jina is licensed under the Apache License, Version 2.0. [See LICENSE for the full license text.](LICENSE)

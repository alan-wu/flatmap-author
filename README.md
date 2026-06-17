# A Container Environment for Flatmap Authors


## Prerequisites:

* [Docker](https://www.docker.com/), [Rancher Desktop](https://rancherdesktop.io/) or a similar container environment. This could be a CLI container builder with a separate virtual machine runner, for example [Colima](https://colima.run/).

### For macOS:

* Install Colima:
    ```
    brew install colima
    ```

* Then edit `~/.colima/default/colima.yaml` and increase `memory` to least 4G; optionally increase `cpu`.

* Use Docker's CLI:
    ```
    brew install docker docker-buildx docker-compose
    brew link docker
    ```

* Edit `~/.docker/config.json` and insert:
    ```
    "cliPluginsExtraDirs": [
        "/opt/homebrew/lib/docker/cli-plugins"
    ]
    ```

---

## Installation:

* Clone this repository and change into it.
* In the `./build` sub-directory:

    ```
    docker buildx build -t flatmap .
    ```

---

## Running:

* Environment variables are used for runtime parameters and must be set before services are started:
    * `SCICRUNCH_API_KEY` should be set to be a valid SciCrunch API key.
    * `FLATMAP_SOURCE_ROOT` needs to be set to a directory path on the host that includes the source directories of maps which will be made.
    * `FLATMAP_SERVER_PORT` is optional. It specifies the port to access the bundled map server, and defailts to `8000`.

### Starting services:

* In the `./run` directory:

    ```
    docker compose up
    ```

---

## Generating maps:

* Open a different terminal session to that running the `flatmap` services.
* Ensure the `SCICRUNCH_API_KEY` and `FLATMAP_SOURCE_ROOT` environment variables are set.
* In the `./run` directory:

    ```
    python runmaker.py --source SOURCE_MANIFEST_PATH [OTHER OPTIONS]
    ```
* The `SOURCE_MANIFEST_PATH` must be under the `FLATMAP_SOURCE_ROOT` directory tree.
* Due to issues with `git` accessing host files from a container, if the map's sources are part of a git-submodule then the parent repository must also be under the `FLATMAP_SOURCE_ROOT` tree.
* The `python` above can be the system Python -- the `runmaker.py` wrapper uses no third-party packages and doesn't have to be installed into a virtual Python environment.
* `python runmaker.py --help` will show `mapmaker`'s runtime help. All options can be used, although `--output` is ignored.

---

## Sample maps

### Test flatmap:

The [test flatmap](https://github.com/AnatomicMaps/test-flatmap) can be made by referencing its GitHub repository when running `runmaker.py`:

```
python runmaker.py \
        --source https://github.com/AnatomicMaps/test-flatmap \
        --manifest manifest.json \
        --background-tiles
```

Alternatively the repository and be cloned, into the `FLATMAP_SOURCE_ROOT` directory tree and made with:

```
python runmaker.py \
        --source /PATH/TO/test-flatmap/manifest.json \
        --background-tiles
```

(assuming `/PATH/TO` is under the `FLATMAP_SOURCE_ROOT` directory).

### Functional connectivity flatmap:

The the published version of the [simple blood volume model on PMR](https://models.physiomeproject.org/e/d77) can be made by cloning its repository and running:

```
python3 runmaker.py \
        --source /PATH/TO/CLONE/map/BloodVolumeControl.manifest.json \
        --background-tiles
```

(again assuming `/PATH/TO` is under the `FLATMAP_SOURCE_ROOT` directory).

---

## Viewing maps:

* Point a web browser at http://localhost:8000/viewer, substituting the value of `FLATMAP_SERVER_PORT` for `8000` if it was set above.

---

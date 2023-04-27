## JupyterLab for Data Science and Knowledge Graphs

[![Publish Jupyter Latest image](https://github.com/luanpaschoal/jupyterlab/actions/workflows/docker-cpu.yml/badge.svg)](https://github.com/luanpaschoal/jupyterlab/actions/workflows/docker-cpu.yml)

JupyterLab image with VisualStudio Code server integrated, based on the [jupyter/docker-stacks](https://github.com/jupyter/docker-stacks) scipy image, with additional packages and kernels installed for data science and knowledge graphs.

![Screenshot](/icons/screenshot.png)

## 🔋 Features

List of features for the images available running on CPU

### `ghcr.io/luanpaschoal/jupyterlab:latest`

This is the base image with useful interfaces and libraries for data science preinstalled:

📋️ **VisualStudio Code** server is installed, and accessible from the JupyterLab Launcher

🐍 **Python 3.8** with notebook kernel supporting autocomplete and suggestions ([jupyterlab-lsp](https://github.com/krassowski/jupyterlab-lsp))

☕️ **Java OpenJDK 11** with [IJava](https://github.com/SpencerPark/IJava) notebook kernel

🐍 **Conda** and mamba are installed, each conda environment created will add a new option to create a notebook using this environment in the JupyterLab Launcher (with `nb_conda_kernels`). You can create environments using different version of Python if necessary.

🧑‍💻 **ZSH** is used by default for the JupyterLab and VisualStudio Code terminals

The following JupyterLab extensions are also installed: [jupyterlab-git](https://github.com/jupyterlab/jupyterlab-git), [jupyterlab-system-monitor](https://github.com/jtpio/jupyterlab-system-monitor), [jupyter_bokeh](https://github.com/bokeh/jupyter_bokeh), [plotly](https://github.com/plotly/plotly.py), [jupyterlab-spreadsheet](https://github.com/quigleyj97/jupyterlab-spreadsheet), [jupyterlab-drawio](https://github.com/QuantStack/jupyterlab-drawio).

#### Automatically install your code and dependencies

With those docker images, you can optionally provide the URL to a git repository to be automatically cloned in the workspace at the start of the container using the environment variable `GIT_URL`

The following files will be automatically installed if they are present at the root of the provided Git repository:

* The conda environment described in `environment.yml` will be installed, make sure you added `ipykernel` and `nb_conda_kernels` to the `environment.yml` to be able to easily start notebooks using this environment from the JupyterLab Launcher page. See [this repository as example]( https://github.com/MaastrichtU-IDS/dsri-demo).
* The python packages in `requirements.txt` will be installed with `pip`
* The debian packages in `packages.txt` will be installed with `apt-get`
* The JupyterLab extensions in `extensions.txt` will be installed with `jupyter labextension`

You can also create a conda environment from a file in a running JupyterLab (we use `mamba` which is like `conda` but faster):

```bash
mamba env create -f environment.yml
```

You'll need to wait a minute before the new conda environment becomes available on the JupyterLab Launcher page.

### 📝 Extend a CPU image

The easiest way to build a custom image is to extend the [existing images](https://github.com/MaastrichtU-IDS/jupyterlab).

For notebooks running on CPU, we use images from the official [jupyter/docker-stacks](jupyter/docker-stacks), which run as non root user. So you will need to make sure the folders permissions are properly set for the notebook user.

Here is an example `Dockerfile` to extend [`ghcr.io/maastrichtu-ids/jupyterlab:latest`](https://github.com/MaastrichtU-IDS/jupyterlab/blob/main/Dockerfile):

```dockerfile
FROM ghcr.io/maastrichtu-ids/jupyterlab:latest
# Change to root user to install packages requiring admin privileges:
USER root
RUN apt-get update && \
    apt-get install -y vim
RUN fix-permissions /home/$NB_USER
# Switch back to the notebook user for other packages:
USER ${NB_UID}
RUN mamba install -c defaults -y rstudio
RUN pip install jupyter-rsession-proxy
```

> For docker image that are not based on the jupyter/docker-stack, such as the GPU images, you the root user is used by default. See at the further in this README for more information on how to extend GPU images.

### 🐳 Run a CPU image with Docker

For the `ghcr.io/maastrichtu-ids/jupyterlab:latest` image volumes should be mounted into `/home/jovyan/work` folder.

This command will start JupyterLab as `jovyan` user with `sudo` privileges, use `JUPYTER_TOKEN` to define your password:

```bash
docker run --rm -it --user root -p 8888:8888 -e GRANT_SUDO=yes -e JUPYTER_TOKEN=password -v $(pwd)/data:/home/jovyan/work ghcr.io/maastrichtu-ids/jupyterlab
```

> You should now be able to install anything in the JupyterLab container, try:
>
> ```bash
> sudo apt-get update
> ```

You can check the `docker-compose.yml` file to run it easily with Docker Compose.

Run with a restricted `jovyan` user, without `sudo` privileges:

```bash
docker run --rm -it --user $(id -u) -p 8888:8888 -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS='-R' -e JUPYTER_TOKEN=password -v $(pwd)/data:/home/jovyan/work ghcr.io/maastrichtu-ids/jupyterlab:latest
```

> ⚠️ Potential permission issue when running locally. The official [jupyter/docker-stacks](jupyter/docker-stacks) images use the `jovyan` user by default which does not grant admin rights (`sudo`). This can cause issues when writing to the shared volumes, to fix it you can change the owner of the folder, or start JupyterLab as root user.
>
> To create the folder with the right permissions, replace `1000:100` by your username:group if necessary and run:
>
> ```bash
>mkdir -p data/
> sudo chown -R 1000:100 data/
> ```
>

### 📦 Build CPU images

Instructions to build the various image aiming to run on CPU.

#### JupyterLab for Data Science

This repository contains multiple folders with `Dockerfile` to build various flavor of JupyterLab for Data Science.

With Python 3.8, conda integration, VisualStudio Code, Java and SPARQL kernels

Build:

```bash
docker build -t ghcr.io/luanpaschoal/jupyterlab .
```

Run:

```bash
docker run --rm -it --user root -p 8888:8888 -e JUPYTER_TOKEN=password -v $(pwd)/data:/home/jovyan/work ghcr.io/luanpaschoal/jupyterlab
```

Push:

```bash
docker push ghcr.io/luanpaschoal/jupyterlab
```

## ☁️ Deploy on Kubernetes and OpenShift

This image is compatible with [OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift) and [OKD](https://www.okd.io) security constraints to **run as non root user**.

We recommend to use this [Helm](https://helm.sh/) chart to deploy these JupyterLab images on Kubernetes or OpenShift: https://artifacthub.io/packages/helm/dsri-helm-charts/jupyterlab

## 🕊️ Contribute to this repository

Choose which image fits your need: latest, tensorflow, cuda, pytorch, freesurfer, python2.7...

1. Fork this repository.
2. Clone the forked repository
3. Edit the `Dockerfile` for the image you want to improve. Preferably use `mamba` or `conda` to install new packages, you can also install with `apt-get` (need to run as root or with `sudo`) and `pip`

4. Go to the folder and rebuild the `Dockerfile`:

```bash
docker build -t jupyterlab -f Dockerfile .
```

5. Run the docker image built on http://localhost:8888 to test it

```bash
docker run -it --rm -p 8888:8888 -e JUPYTER_TOKEN=yourpassword jupyterlab
```

If the built Docker image works well feel free to send a pull request to get your changes merged to the main repository and integrated in the corresponding published Docker image.

You can check the size of the image built in MB:

```bash
expr $(docker image inspect ghcr.io/luanpaschoal/jupyterlab:latest --format='{{.Size}}') / 1000000
```


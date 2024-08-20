
# 🧑‍💻 Installation

How to install the Oxen client, server, or python package. If you are a developer, you will want to [build from source](build_and_run.md). If you are flying by and learning Oxen you can install the python package or the command line tool from the [GitHub releases page](https://github.com/Oxen-AI/Oxen/releases).

## 💻 Command Line Tools

The Oxen client can be installed via [homebrew](https://brew.sh/) or by downloading the relevant binaries for Linux or Windows.

You can find the source code for the client [here](https://github.com/Oxen-AI/Oxen) and can also build for source for your platform. The continuous integration pipeline will build binaries for each release in [this repository](https://github.com/Oxen-AI/Oxen).

### Mac

```bash
brew tap Oxen-AI/oxen
```

```bash
brew install oxen
```

### Ubuntu Latest

Check the [GitHub releases page](https://github.com/Oxen-AI/Oxen/releases) for the latest version of the client and server.

```bash
wget https://github.com/Oxen-AI/Oxen/releases/latest/download/oxen-ubuntu-latest.deb
```

```bash
sudo dpkg -i oxen-ubuntu-latest.deb
```

### Ubuntu 20.04

```bash
wget https://github.com/Oxen-AI/Oxen/releases/latest/download/oxen-ubuntu-20.04.deb
```

```bash
sudo dpkg -i oxen-ubuntu-20.04.deb
```

### Windows

```bash
wget https://github.com/Oxen-AI/Oxen/releases/latest/download/oxen.exe
```

### Other Linux

Binaries are coming for other Linux distributions in the future. [In the meanwhile, you can build from source.](#building-from-source)

## 🌎 Server Install

The Oxen server binary can be deployed where ever you want to store and backup your data. It is an HTTP server that the client communicates with to enable collaboration.

### Mac

```bash
brew tap Oxen-AI/oxen-server
```

```bash
brew install oxen-server
```

### Docker

```bash
wget https://github.com/Oxen-AI/Oxen/releases/latest/download/oxen-server-docker.tar
```

```bash
docker load < oxen-server-docker.tar
```

```bash
docker run -d -v /var/oxen/data:/var/oxen/data -p 80:3001 oxen/oxen-server:latest
```

### Ubuntu Latest

```bash
wget https://github.com/Oxen-AI/Oxen/releases/latest/download/oxen-server-ubuntu-latest.deb
```

```bash
sudo dpkg -i oxen-server-ubuntu-latest.deb
```

### Ubuntu 20.04

```bash
wget https://github.com/Oxen-AI/Oxen/releases/latest/download/oxen-server-ubuntu-20.04.deb
```

```bash
sudo dpkg -i oxen-server-ubuntu-20.04.deb
```

### Windows

```bash
wget https://github.com/Oxen-AI/Oxen/releases/latest/download/oxen-server.exe
```

To get up and running using the client and server, you can follow the [getting started docs](https://github.com/Oxen-AI/oxen-release).

## 🐍 Python Package

```bash
$ pip install oxenai
```

Note that this will only install the Python library and not the command line tool.

### Installing Oxen through Jupyter Notebooks or Google Colab

Create and run this cell:
```python
!pip install oxenai
```

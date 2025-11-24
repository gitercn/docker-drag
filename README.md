# docker-drag
This repository contains a Python script for interacting with Docker Hub or other registries, without needing the Docker client itself.

It relies on the Docker registry [HTTPS API v2](https://docs.docker.com/registry/spec/api/).

## Usage

### Basic Usage

To pull a standard Docker image, provide the image name as an argument. The script will download the image and package it as a `.tar` file.

```shell
# Pull the hello-world image
python docker_pull.py hello-world

# Pull a specific version of an official image
python docker_pull.py mysql/mysql-server:8.0
python docker_pull.py protopie/enterprise-onpremises:api-15.8.3

# Pull from a different registry
python docker_pull.py mcr.microsoft.com/mssql-tools

# Pull an image by its digest
python docker_pull.py consul@sha256:6ba4bfe1449ad8ac5a76cb29b6c3ff54489477a23786afb61ae30fb3b1ac0ae9
```

After the image has been downloaded, you can load it into Docker using `docker load`:
```shell
docker load -i <image_name>.tar
docker run -it <image_name>
```

### Handling Multi-Architecture Images

Many modern Docker images support multiple CPU architectures (e.g., `amd64`, `arm64`). This script allows you to select which architecture to pull.

**1. List Available Platforms**

If you run the script on a multi-architecture image without specifying a platform, it will list all available platforms and the corresponding `--platform` argument to use.

```shell
python docker_pull.py hello-world
```

**Example Output:**
```
[+] This is a multi-architecture image. Please specify a platform using the --platform argument.
[i] Available platforms are:
  --platform linux/amd64                # os: linux, architecture: amd64 (digest: sha256:...)
  --platform linux/arm/v5               # os: linux, architecture: arm, variant: v5 (digest: sha256:...)
  --platform linux/arm/v7               # os: linux, architecture: arm, variant: v7 (digest: sha256:...)
  --platform linux/arm64/v8             # os: linux, architecture: arm64, variant: v8 (digest: sha256:...)
  --platform windows/amd64              # os: windows, architecture: amd64, os.version: ... (digest: sha256:...)
```

**2. Pull a Specific Platform**

Use the `--platform` flag with the desired platform string from the list above.

```shell
# Pull the linux/arm64 version of hello-world
python docker_pull.py hello-world --platform linux/arm64/v8
```

The script will then download the image for the specified architecture.

<p align="center">
  <img src="https://user-images.githubusercontent.com/26483750/77766160-8da6f080-703f-11ea-953c-fd69978cb3bf.gif">
</p>


## Limitations
- Only support v2 manifests: some registries, like quay.io which only uses v1 manifests, may not work.

## Well known bugs
2 open bugs which shouldn't affect the efficiency of the script nor the pulled image:
- Unicode content (for example `\u003c`) gets automatically decoded by `json.loads()` which differs from the original Docker client behaviour (`\u003c` should not be decoded when creating the TAR file). This is due to the json Python library automatically converting string to unicode.
- Fake layers ID are not calculated the same way than Docker client does (I don't know yet how layer hashes are generated, but it seems deterministic and based on the client)
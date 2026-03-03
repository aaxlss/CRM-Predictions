# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.245.2/containers/python-3
FROM mcr.microsoft.com/devcontainers/python:0-3.10-bullseye

# Copy requirements.txt
COPY .devcontainer/requirements.txt /tmp/pip-tmp/

# [Choice] Kept install requirements from requirements.txt so that it runs in the container, but commented out the sections applicable to the Dockerfile
# RUN pip3 --disable-pip-version-check --no-cache-dir install -r /tmp/pip-tmp/requirements.txt 
#    && rm -rf /tmp/pip-tmp

# [Optional] If your requirements rarely change, uncomment this section to add them to the image.
# COPY requirements.txt /tmp/pip-tmp/
RUN pip3 --disable-pip-version-check --no-cache-dir install -r /tmp/pip-tmp/requirements.txt
#
# RUN rm -rf /tmp/pip-tmp

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive 
#     && apt-get -y install --no-install-recommends <your-package-list-here>

RUN pip install shap
RUN pip install imbalanced-learn
# Using a TensorFlow base image with GPU support and Jupyter pre-installed
FROM tensorflow/tensorflow:latest-gpu-jupyter

# Copy requirements.txt
COPY .devcontainer/requirements.txt /tmp/pip-tmp/

# Install packages from requirements.txt
RUN pip3 --disable-pip-version-check --no-cache-dir install -r /tmp/pip-tmp/requirements.txt \
    && rm -rf /tmp/pip-tmp

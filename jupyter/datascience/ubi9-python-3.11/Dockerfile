ARG BASE_IMAGE
FROM ${BASE_IMAGE}

ARG SOURCE_CODE=jupyter/datascience/ubi9-python-3.11

LABEL name="odh-notebook-jupyter-datascience-ubi9-python-3.11" \
    summary="Jupyter data science notebook image for ODH notebooks" \
    description="Jupyter data science notebook image with base Python 3.11 builder image based on UBI9 for ODH notebooks" \
    io.k8s.display-name="Jupyter data science notebook image for ODH notebooks" \
    io.k8s.description="Jupyter data science notebook image with base Python 3.11 builder image based on UBI9 for ODH notebooks" \
    authoritative-source-url="https://github.com/opendatahub-io/notebooks" \
    io.openshift.build.commit.ref="main" \
    io.openshift.build.source-location="https://github.com/opendatahub-io/notebooks/tree/main/jupyter/datascience/ubi9-python-3.11" \
    io.openshift.build.image="quay.io/opendatahub/workbench-images:jupyter-datascience-ubi9-python-3.11"

WORKDIR /opt/app-root/bin

# Install Python packages and Jupyterlab extensions from Pipfile.lock
COPY ${SOURCE_CODE}/Pipfile.lock ./

# Copy Elyra setup to utils so that it's sourced at startup
COPY ${SOURCE_CODE}/setup-elyra.sh ./utils/

RUN echo "Installing softwares and packages" && \
    micropipenv install && \
    rm -f ./Pipfile.lock && \
    # Fix permissions to support pip in Openshift environments \
    chmod -R g+w /opt/app-root/lib/python3.11/site-packages && \
    fix-permissions /opt/app-root -P

COPY ${SOURCE_CODE}/utils ./utils/

# OS Packages needs to be installed as root
USER root

# Install usefull OS packages
RUN dnf install -y jq unixODBC postgresql git-lfs libsndfile && dnf clean all && rm -rf /var/cache/yum

# Disable announcement plugin of jupyterlab
RUN jupyter labextension disable "@jupyterlab/apputils-extension:announcements"

# Install MongoDB Client, We need a special repo for MongoDB as they do their own distribution
COPY ${SOURCE_CODE}/mongodb-org-6.0.repo-x86_64 /etc/yum.repos.d/mongodb-org-6.0.repo

RUN dnf install -y mongocli && dnf clean all && rm -rf /var/cache/yum

# Install MSSQL Client, We need a special repo for MSSQL as they do their own distribution
COPY ${SOURCE_CODE}/mssql-2022.repo-x86_64 /etc/yum.repos.d/mssql-2022.repo

RUN ACCEPT_EULA=Y dnf install -y mssql-tools18 unixODBC-devel && dnf clean all && rm -rf /var/cache/yum

ENV PATH="$PATH:/opt/mssql-tools18/bin"

# Other apps and tools installed as default user
USER 1001

# setup path for runtime configuration
RUN mkdir /opt/app-root/runtimes && \
    # Replace Notebook's launcher, "(ipykernel)" with Python's version 3.x.y \
    sed -i -e "s/Python.*/$(python --version | cut -d '.' -f-2)\",/" /opt/app-root/share/jupyter/kernels/python3/kernel.json && \
    # Remove default Elyra runtime-images \
    rm /opt/app-root/share/jupyter/metadata/runtime-images/*.json && \
    # Fix permissions to support pip in Openshift environments \
    chmod -R g+w /opt/app-root/lib/python3.11/site-packages && \
    fix-permissions /opt/app-root -P

# Copy Elyra runtime-images definitions and set the version
COPY ${SOURCE_CODE}/runtime-images/ /opt/app-root/share/jupyter/metadata/runtime-images/

WORKDIR /opt/app-root/src

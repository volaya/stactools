FROM ubuntu:18.04

# Setup timezone info
ENV TZ=UTC

ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update && apt-get install -y software-properties-common

RUN add-apt-repository ppa:ubuntugis/ppa && \
    apt-get update && \
    apt-get install -y wget=1.* git=1:2.* python-protobuf=3.* python3-tk=3.* \
                       build-essential python3-dev python3-numpy \
                       jq=1.5* unzip curl \
                       libsqlite3-dev=3.22.* zlib1g-dev=1:1.2.* \
                       libhdf4-dev && \
    apt-get autoremove && apt-get autoclean && apt-get clean

ENV CURL_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt

# Install Python 3.8
RUN wget -q -O ~/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-4.7.12.1-Linux-x86_64.sh && \
     chmod +x ~/miniconda.sh && \
     ~/miniconda.sh -b -p /opt/conda && \
     rm ~/miniconda.sh
ENV PATH /opt/conda/bin:$PATH
ENV LD_LIBRARY_PATH /opt/conda/lib/:$LD_LIBRARY_PATH
RUN conda install -y python=3.8
RUN python -m pip install --upgrade pip

# Install GDAL
RUN conda install -y -c conda-forge gdal=3.2.0

# Core
COPY stactools_core/requirements.txt /tmp/core/requirements.txt
RUN pip install -r /tmp/core/requirements.txt

# Install dependencies for building rasterio from source.
# The rasterio wheels do not include the HDF-EOS driver
# for GDAL that we need for reading aster and corine files,
# so we build it from source. We disable pip's build isolation,
# since rasterio's pyproject.toml doesn't currently pin
# the NumPy version and so will use the newest available. If
# we build rasterio against a newer version of NumPy, we'll
# be unable to import it in *this* environment, which might
# already have NumPy installed.
#
# Build dependencies for aster
RUN pip install setuptools cython numpy

# CLI
COPY stactools_cli/requirements.txt /tmp/cli/requirements.txt
RUN pip install --no-build-isolation -r /tmp/cli/requirements.txt

# Aster
COPY stactools_aster/requirements.txt /tmp/aster/requirements.txt
RUN pip install --no-build-isolation -r /tmp/aster/requirements.txt

# Corine
COPY stactools_corine/requirements.txt /tmp/corine/requirements.txt
RUN pip install -r /tmp/corine/requirements.txt

# Landsat
COPY stactools_landsat/requirements.txt /tmp/landsat/requirements.txt
RUN pip install -r /tmp/landsat/requirements.txt

# Planet
COPY stactools_planet/requirements.txt /tmp/planet/requirements.txt
RUN pip install -r /tmp/planet/requirements.txt

# Jupyter
RUN pip install jupyter==1.0.0

WORKDIR /opt/src

ENV PYTHONPATH=/opt/src:$PYTHONPATH

COPY stactools /opt/src/stactools

ENTRYPOINT [ "python", "-m", "stactools.cli" ]

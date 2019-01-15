FROM buildpack-deps:bionic-scm

# Set up common env variables
ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV DEBIAN_FRONTEND=noninteractive

RUN adduser --disabled-password --gecos "Default Jupyter user" jovyan

RUN apt-get update --yes
RUN apt-get install --yes \
		python3.6 \
		python3.6-venv \
		python3.6-dev \
		tar \
		vim \
		nodejs \
		locales

# Other packages for user convenience and Data100 usage
# Install these without 'recommended' packages to keep image smaller.
RUN apt-get install --yes --no-install-recommends \
		build-essential \
		ca-certificates \
		curl \
		default-jdk \
		emacs-nox \
		git \
		htop \
		less \
		man \
		mc \
		nano \
		openssh-client \
		postgresql-client \
		screen \
		tar \
		tmux \
		wget

RUN echo "${LC_ALL} UTF-8" > /etc/locale.gen && \
	locale-gen

# for nbconvert
RUN apt-get install --yes \
		# for nbconvert
		pandoc \
		texlive-xetex \
		texlive-fonts-recommended \
		texlive-generic-recommended

# Keep image size at a minimum
RUN apt-get clean

ENV CONDA_PREFIX /srv/conda
ENV PATH ${CONDA_PREFIX}/bin:$PATH
RUN install -d -o jovyan -g jovyan ${CONDA_PREFIX}

WORKDIR /home/jovyan

USER jovyan

####################################################################
# Download, install and configure the Conda environment

RUN curl -o /tmp/miniconda.sh \
	https://repo.continuum.io/miniconda/Miniconda3-4.4.10-Linux-x86_64.sh

# Install miniconda
RUN bash /tmp/miniconda.sh -b -u -p ${CONDA_PREFIX}

RUN conda config --set always_yes yes --set changeps1 no
RUN conda update -q conda

# Encapsulate the environment info into its own yml file (which carries
# the name `data100` in it
COPY environment.yml /tmp/
RUN conda env create -q -f /tmp/environment.yml

# We modify the path directly since the `source activate data100`
# environment won't be preserved here.
ENV PATH ${CONDA_PREFIX}/envs/data100/bin:$PATH

# Set bash as shell in terminado. Disable history.
ADD jupyter_notebook_config.py ${CONDA_PREFIX}/envs/data100/etc/jupyter/
ADD ipython_config.py ${CONDA_PREFIX}/envs/data100/etc/ipython/

RUN pip install --no-cache-dir jupyterhub==0.9.2

RUN pip install psycopg2==2.7.6.1

# Installed in conda environment
RUN jupyter serverextension enable --sys-prefix --py jupyterlab

#RUN pip install nbgrader==0.5.4
RUN pip install git+git://github.com/berkeley-dsep-infra/nbgrader.git@7aba39a
RUN jupyter nbextension install --sys-prefix --py nbgrader
RUN jupyter nbextension enable --sys-prefix --py nbgrader
RUN jupyter serverextension enable --sys-prefix --py nbgrader

# For dask dashboard
RUN pip install nbserverproxy==0.8.8
RUN jupyter serverextension enable --sys-prefix --py nbserverproxy

# Disabling the autosavetime interval
# Install nb extensions
RUN pip install jupyter_contrib_nbextensions
# We install, ignoring running servers
# RUN jupyter contrib nbextensions install --sys-prefix --skip-running-check
# Enable autosavetime which will by default disable autosave
# RUN jupyter nbextension enable autosavetime/main

# Install nbgitpuller
RUN pip install --no-cache-dir nbgitpuller==0.6.1

# Install nbzip
RUN pip install --no-cache-dir nbzip==0.1.0

RUN jupyter serverextension enable  --sys-prefix --py nbzip
RUN jupyter nbextension install     --sys-prefix --py nbzip
RUN jupyter nbextension enable      --sys-prefix --py nbzip

RUN pip install ray==0.6.1

# Useful for debugging any issues with conda
RUN conda info -a

# Make JupyterHub ports visible
EXPOSE 8888

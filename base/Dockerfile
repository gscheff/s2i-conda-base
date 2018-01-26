# This image provides a Python 3.6 environment you can use to run your Python
# applications.
FROM centos/s2i-base-centos7

ENV PATH=$HOME/.local/bin/:$PATH \
    PYTHONUNBUFFERED=1 \
    PYTHONIOENCODING=UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    PIP_NO_CACHE_DIR=off

RUN yum install -y centos-release-scl-rh && \
    yum-config-manager --enable centos-sclo-rh-testing && \
    INSTALL_PKGS="rh-python36 rh-python36-python-devel rh-python36-python-setuptools rh-python36-python-pip \
	 nss_wrapper httpd24 httpd24-httpd-devel httpd24-mod_ssl httpd24-mod_auth_kerb httpd24-mod_ldap \
         httpd24-mod_session atlas-devel gcc-gfortran libffi-devel libtool-ltdl enchant" && \
    yum install -y --setopt=tsflags=nodocs --enablerepo=centosplus $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    # Remove centos-logos (httpd dependency, ~20MB of graphics) to keep image
    # size smaller.
    rpm -e --nodeps centos-logos && \
    yum clean all -y

# Install Python 3 from miniconda
ADD https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh miniconda.sh
RUN bash miniconda.sh -b -p ${APP_ROOT}/miniconda
RUN rm miniconda.sh

# keep conda in user dir, so can do conda install in notebook
ENV PATH="${APP_ROOT}/miniconda/bin:$PATH"

# - Create a Python virtual environment for use by any application to avoid
#   potential conflicts with Python packages preinstalled in the main Python
#   installation.
RUN conda update conda \
  && conda config --set always_yes yes \
  && conda install -mn venv python

# Install pydata stack
#RUN conda install -mn venv notebook psutil numpy pandas scikit-learn statsmodels pip numba \
#  scikit-image datashader holoviews nomkl matplotlib lz4 tornado
#RUN conda install -mn venv -c conda-forge fastparquet s3fs zict python-blosc cytoolz \
#  dask distributed dask-searchcv gcsfs

# clean conda caches
RUN conda clean -tipsy

# pip install
#RUN pip install git+https://github.com/dask/dask-glm.git --no-deps\
# && pip install graphviz

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH.
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Copy extra files to the image.
COPY ./root/ /

# - In order to drop the root user, we have to make some directories world
#   writable as OpenShift default security model is to run the container
#   under random UID.
RUN chown -R 1001:0 ${APP_ROOT} && \
    fix-permissions ${APP_ROOT} && \
    rpm-file-permissions
USER 1001

ENV SUMMARY="Platform for building and running Python $PYTHON_VERSION applications" \
    DESCRIPTION="Python $PYTHON_VERSION available as docker container is a base platform for \
building and running various Python $PYTHON_VERSION applications and frameworks. \
Python is an easy to learn, powerful programming language. It has efficient high-level \
data structures and a simple but effective approach to object-oriented programming. \
Python's elegant syntax and dynamic typing, together with its interpreted nature, \
make it an ideal language for scripting and rapid application development in many areas \
on most platforms."

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="Python 3.6" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,python,python36" \
      name="gscheff/s2i_python" \
      version="1.0" \
      usage="s2i build https://github.com/gscheff/s2i_python.git --context-dir=/example/setup-test-app/ gscheff/s2i_python python-sample-app" \
      maintainer="Gerhard Scheffler"

EXPOSE 8080

# Set the default CMD to print the usage of the language image.
CMD $STI_SCRIPTS_PATH/usage


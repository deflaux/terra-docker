FROM us.gcr.io/broad-dsp-gcr-public/terra-jupyter-base:0.0.19
USER root
# This makes it so pip runs as root, not the user.
ENV PIP_USER=false

RUN apt-get update && apt-get install -yq --no-install-recommends \
  python-tk \
  tk-dev \
  libssl-dev \
  xz-utils \
  libhdf5-dev \
  openssl \
  make \
  liblzo2-dev \
  zlib1g-dev \
  libz-dev \
  libcurl4-openssl-dev \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

ENV HTSLIB_CONFIGURE_OPTIONS="--enable-gcs"

# Dev note: in general, do not pin Python packages to any particular version. Depend on the smoke tests
# to help us identify any package incompatibilties.
# 
# If we find that we do need to pin a package version, be sure to:
# 1) Add a comment saying what needs to be true for us to remove the pin.
#    (e.g. link to an issue and put the details there)
# 2) If the smoke tests did not show the problem, add a new test case to improve
#    test coverage for the identified problem.
RUN pip3 -V \
 # Get rid of "WARNING: The directory '/home/jupyter-user/.cache/pip' or its parent directory is not owned or is not writable by the current user. The cache has been disabled."
 && chmod a+w /home/jupyter-user/.cache/pip \
 && pip3 install --upgrade pip \
 && pip3 install --upgrade \
   biopython \
   bleach \
   bokeh \
   bx-python \
   certifi \
   cycler \
   Cython \
   fastinterval \
   ggplot \
   google-cloud-bigquery \
   google-cloud-bigquery-datatransfer \
   google-cloud-datastore \
   google-cloud-resource-manager \
   google-resumable-media \
   google-cloud-storage \
   h5py \
   html5lib \
   intel-openmp \
   joblib \
   keras \
   markdown \
   matplotlib-venn \
   mkl \
   pandas-gbq \
   pandas-profiling \
   patsy \
   pdoc3 \
   plotnine \
   py4j \
   pyfasta \
   pymc3 \
   pyparsing \
   pysam --no-binary pysam \
   python-dateutil \
   python-lzo \
   pytz \
   pyvcf \
   pyyaml \
   readline \
   scikit-learn \
   scipy \
   seaborn \
   setuptools \
   statsmodels \
   # Use the cpu version of Tensorflow to eliminate the warnings about absent gpus on the Cloud Runtime.
   tensorflow_cpu \
   theano \
   tqdm \
   werkzeug \
   wheel \
 && pip3 uninstall -y \
   # Remove this after https://broadworkbench.atlassian.net/browse/CA-1179
   # As of release [google-cloud-bigquery 1.26.0 (2020-07-20)](https://github.com/googleapis/python-bigquery/blob/master/CHANGELOG.md#1260-2020-07-20)
   # the BigQuery Python client uses the BigQuery Storage client by default.
   # This currently causes an error on Terra Cloud Runtimes `the user does not have 'bigquery.readsessions.create' permission for '<Terra billing project id>'`.
   # To work-around this uninstall the dependency so that flag `--use_rest_api` can be used with `%%bigquery` to use the older, slower mechanism for data transfer.
  google-cloud-bigquery-storage  

ENV USER jupyter-user
USER $USER
# We want pip to install into the user's dir when the notebook is running
ENV PIP_USER=true

# Note: this entrypoint is provided for running Jupyter independently of Leonardo.
# When Leonardo deploys this image onto a cluster, the entrypoint is overwritten to enable
# additional setup inside the container before execution.  Jupyter execution occurs when the
# init-actions.sh script uses 'docker exec' to call run-jupyter.sh.
ENTRYPOINT ["/usr/local/bin/jupyter", "notebook"]

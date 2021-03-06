# adapted from https://hub.docker.com/r/jupyter/base-notebook/ AKA https://github.com/jupyter/docker-stacks/tree/master/base-notebook

FROM gcr.io/deeplearning-platform-release/base-cu110:latest
USER root

#######################
# Prerequisites
#######################
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -yq --no-install-recommends \

    # gnupg requirement
    dirmngr \
    gnupg \

    # curl requirement
    ca-certificates \

    # useful utilities for debugging within the docker
    nano \
    procps \

    lsb-release \
    gcc-6 \
    gcc-8 \

    # python requirements
    checkinstall \
    build-essential \
    zlib1g-dev \

    # pip requirements
    libssl1.0-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    llvm \
    libncurses5-dev \
    libncursesw5-dev \
    tk-dev \
    libffi-dev \
    liblzma-dev \
    python-openssl \

    # install script requirements
    sudo \
    locales \

    # for ssh-agent and ssh-add
    keychain \

    # openjdk 11
    default-jre \
    default-jdk \

 # Uncomment en_US.UTF-8 for inclusion in generation
 && sed -i 's/^# *\(en_US.UTF-8\)/\1/' /etc/locale.gen \
 # Generate locale
 && locale-gen \

 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

ENV LC_ALL en_US.UTF-8

#######################
# Python / Jupyter
#######################

ENV USER jupyter
RUN usermod -g users jupyter
ENV HOME /home/$USER

# The welder uid is consistent with the Welder docker definition here:
#  https://github.com/DataBiosphere/welder/blob/master/project/Settings.scala
# Adding welder-user to the Jupyter container isn't strictly required, but it makes welder-added
# files display nicer when viewed in a terminal.
ENV WELDER_USER welder-user
ENV WELDER_UID 1001
RUN useradd -m -s /bin/bash -N -u $WELDER_UID $WELDER_USER

# ensure this matches c.NotebookApp.port in jupyter_notebook_config.py
ENV JUPYTER_PORT 8000
ENV JUPYTER_HOME /etc/jupyter
ENV USER_INSTALL_DIR /home/jupyter/notebooks/packages

# Prepend existing PYTHONPATH if it's not empty; otherwise prepend it
ENV PYTHONPATH ${PYTHONPATH:+$PYTHONPATH:}$JUPYTER_HOME/custom:$USER_INSTALL_DIR
ENV PATH ${PATH:+$PATH:}$USER_INSTALL_DIR/bin

RUN pip3 -V \
 # When we upgraded from jupyter 5.7.8 to 6.1.1, we broke terminal button on terra-ui.
 # Hence, make sure to manually test out "launch terminal" button (the button in the green bar next to start and stop buttons)
 # to make sure we don't accidentally break it every time we upgrade notebook version until we figure out an automation test for this
 && pip3 install python-datauri \
 # This is to avoid incompatibility issue, see https://github.com/ipython-contrib/jupyter_contrib_nbextensions/issues/1529
 && conda install "nbconvert<6" \
 && pip3 install jupyter_contrib_nbextensions \
 && pip3 install jupyter_nbextensions_configurator \
 # for jupyter_delocalize.py and jupyter_notebook_config.py
 && pip3 install requests \
 && pip3 install firecloud \
 && pip3 install terra-notebook-utils \
 && pip3 install crcmod \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# tmp hack min-5
# I'm not installing jupyterlab and I can't update init-actions.sh to not access it
RUN mkdir -p /usr/local/share/jupyter/lab

# make pip install to a user directory, instead of a system directory which requires root.
# this is useful so `pip install` commands can be run in the context of a notebook.
ENV PIP_USER=true
# When using PIP_USER=true packages are installed into Python site.USER_BASE, which is '/home/jupyter' for this system.
# Append '/home/jupyter/.local/bin' to PATH
# pip docs: https://pip.pypa.io/en/stable/reference/pip_install/#cmdoption-user
ENV PATH="${PATH}:${HOME}/.local/bin"

# Default terminal directory should be where PD is mounted
RUN echo "cd ${HOME}/notebooks" >> ${HOME}/.bashrc

#######################
# Utilities
#######################

COPY scripts $JUPYTER_HOME/scripts
COPY custom $JUPYTER_HOME/custom
COPY jupyter_notebook_config.py $JUPYTER_HOME

RUN chown -R $USER:users $JUPYTER_HOME \
# Disable nb_conda for now. Consider re-enable in the future
 && jupyter nbextension disable nb_conda --py --sys-prefix \
 && find $JUPYTER_HOME/scripts -name '*.sh' -type f | xargs chmod +x \
 && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 200 \
 && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 100 \
 && $JUPYTER_HOME/scripts/extension/install_jupyter_contrib_nbextensions.sh \
 && mkdir $HOME/.jupyter/custom/ \
 && cp $JUPYTER_HOME/custom/google_sign_in.js $HOME/.jupyter/custom/ \
 && cp $JUPYTER_HOME/custom/extension_entry_jupyter.js $HOME/.jupyter/custom/custom.js \
 && cp $JUPYTER_HOME/custom/safe-mode.js $HOME/.jupyter/custom/ \
 && cp $JUPYTER_HOME/custom/edit-mode.js $HOME/.jupyter/custom/ \
 && mkdir $JUPYTER_HOME/nbconfig \
#  You can get kernel directory by running `jupyter kernelspec list`
 && $JUPYTER_HOME/scripts/kernel/kernelspec.sh $JUPYTER_HOME/scripts/kernel /opt/conda/share/jupyter/kernels \
 && ls -la ${HOME}/.local

USER $USER
WORKDIR $HOME

EXPOSE $JUPYTER_PORT

# Note: this entrypoint is provided for running Jupyter independently of Leonardo.
# When Leonardo deploys this image onto a cluster, the entrypoint is overwritten to enable
# additional setup inside the container before execution.  Jupyter execution occurs when the
# init-actions.sh script uses 'docker exec' to call run-jupyter.sh.
ENTRYPOINT ["/opt/conda/bin/jupyter", "notebook"]

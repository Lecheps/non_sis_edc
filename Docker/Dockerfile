################################################################################################### Based on jupyter datascience notebook
# Adding:
# -> visual studio code
# -> requisites for opera
###################################################################################################
ARG BASE_CONTAINER=jupyter/datascience-notebook:r-3.6.3
FROM $BASE_CONTAINER

LABEL maintainer="José-Luis Guerrero"

USER root


# OS dependencies =================================================================================
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libaio1 \
    curl \
    rpm2cpio \
    cpio && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get purge -y --auto-remove curl rpm2cpio cpio && \
    apt-get clean

# Installation tools
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    gdebi-core \
    software-properties-common && \
    rm -rf /var/lib/apt/lists/*

# Install Rsync
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    rsync && \
    rm -rf /var/lib/apt/lists/*
    
# Install zip utils 
RUN apt-get update && \
    apt-get install -y \
    zip \
    gzip \
    tar && \
    rm -rf /var/lib/apt/lists/*
    
# Install postgresql libraries for psycopg2
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libpq-dev && \
    rm -rf /var/lib/apt/lists/*

#yarn is required to build npm packages and therefore jupyter lab extensions   
RUN conda install -c conda-forge --quiet --yes \
    'yarn' && \
    conda clean --all -f -y && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Visual Studio Code ==============================================================================
# Based on https://github.com/radiant-rstats/docker/blob/master/rsm-msba/Dockerfile
ENV CODE_SERVER="3.5.0"

#RUN mkdir /opt/code-server && \
#    cd /opt/code-server && \
#    wget -qO- https://github.com/cdr/code-server/releases/download/v$CODE_SERVER/code-server-${CODE_SERVER}-linux-x86_64.tar.gz | tar zxvf - --strip-components=1    
 
RUN curl -fsSL https://code-server.dev/install.sh | sh

#RUN systemctl enable --now code-server$USER
 
# Locations to store vscode / code-server settings
ARG CODE_WORKINGDIR="/home/$NB_USER" 
ENV CODE_WORKINGDIR="${CODE_WORKINGDIR}" \
    CODE_USER_DATA_DIR="/home/$NB_USER/.niva_dst/share/code-server" \
    CODE_EXTENSIONS_DIR="/home/code-server/extensions" \
    CODE_BUILTIN_EXTENSIONS_DIR="/opt/code-server/extensions" \
    PATH=/opt/code-server:$PATH

# Make environment variable available from Rstudio
#RUN echo "CODE_EXTENSIONS_DIR=${CODE_EXTENSIONS_DIR}" >> /etc/R/Renviron.site

# Setup for code-server
COPY jupyter_notebook_config.py /etc/jupyter/
COPY images/vscode.svg /opt/code-server/vscode.svg
COPY settings.json $CODE_USER_DATA_DIR/User/settings.json 
RUN mkdir /opt/code-server/extensions/

RUN cd /opt/code-server/extensions/ && \
    wget https://marketplace.visualstudio.com/_apis/public/gallery/publishers/VisualStudioExptTeam/vsextensions/vscodeintellicode/1.2.10/vspackage && mv vspackage vspackage.gz &&\
    gunzip vspackage.gz && mv vspackage VisualStudioExptTeam.vscodeintellicode-1.2.10.vsix && \
    cd $HOME

RUN ls -lh /opt/code-server/extensions
    

# Install VSCode extensions
RUN cd /opt/code-server/extensions/ && \
    code-server --install-extension "mechatroner.rainbow-csv" && \
    code-server --install-extension "vscode-icons-team.vscode-icons" && \
    code-server --install-extension "CoenraadS.bracket-pair-colorizer-2" && \
    code-server --extensions-dir  $CODE_EXTENSIONS_DIR --install-extension "VisualStudioExptTeam.vscodeintellicode-1.2.10.vsix" && \
    code-server --install-extension "ms-vscode.cpptools" && \
    code-server --install-extension "ms-vscode.cmake-tools" && \
    code-server --install-extension "ms-python.python" 

# Give user correct permissions for code-server extensions directory
RUN chown -R $NB_USER:$NB_GID /home/code-server && \
    fix-permissions /home/code-server 

# Python-specific ================================================================================= 
# Additional Python packages from PIP
RUN python -m pip install pip --upgrade --no-cache-dir && \
    rm -rf /tmp/* 
    
RUN python -m pip install --no-cache-dir \
    'beautifulsoup4' \
    'black' \
    'cython' \
    'ipyleaflet' \
    'ipympl' \
    'jupyter-server-proxy' \
    'jupyterlab_code_formatter' \
    'jupyterlab-git' \
    'jupyterlab_sql ' \
    'pandas' \
    'pybind11' \
    'pygments' \
    'scipy' \
    'seaborn' \
    'fabric' \
    'statsmodels' \
    'fabric' \
    'psycopg2' \
    'gmaps' \
    'xlrd' && \
    rm -rf /tmp/* 
   
# JupyterLab extensions ===========================================================================
ENV NODE_OPTIONS="--max-old-space-size=4096"

  
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager && \
    jupyter labextension install @jupyterlab/server-proxy --no-build && \
    jupyter labextension install jupyter-matplotlib --no-build && \
    jupyter labextension install jupyter-leaflet --no-build && \
    jupyter labextension install @jupyterlab/shortcutui --no-build && \
    jupyter labextension install jupyterlab-spreadsheet --no-build && \
    jupyter labextension install @ryantam626/jupyterlab_code_formatter --no-build && \
    jupyter labextension install @jupyterlab/toc --no-build && \
    jupyter labextension install @jupyterlab/git --no-build && \
    jupyter lab clean && \
    jupyter lab build && \
    jupyter lab clean && \
    jlpm cache clean && \
    npm cache clean --force && \
    rm -rf $HOME/.node-gyp && \
    rm -rf $HOME/.local && \
    rm -rf /tmp/*  && \
    fix-permissions $CONDA_DIR $HOME
    
   
RUN jupyter serverextension enable --py --sys-prefix --system jupyterlab_git && \
    jupyter serverextension enable --py --sys-prefix --system jupyterlab_code_formatter && \
    jupyter nbextension enable --py --sys-prefix --system ipyleaflet && \
    jupyter lab build && \
    jupyter lab clean
    
    
# General =========================================================================================

# Git config
RUN git config --global core.editor "nano"
 
EXPOSE 1521

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID

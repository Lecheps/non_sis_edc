ARG BASE_CONTAINER=jupyter/datascience-notebook:41d6b29cd15c
FROM $BASE_CONTAINER

LABEL maintainer="José-Luis Guerrero <jlg@niva.com>"

USER root

RUN conda update conda
RUN conda update -n base conda

#Install R kernel and basic R components
RUN conda install --quiet --yes \
'r-base' 'r-core' 'r-recommended' \
'r-irkernel'

RUN conda clean --all -f -y && \
fix-permissions $CONDA_DIR && \
fix-permissions /home/$NB_USER

#Installing debian packages
RUN apt-get update
RUN apt-get install gcc gfortran software-properties-common

#Installing python packages
RUN conda install --quiet --yes
'fabric' 'psycopg2'

USER $NB_UID

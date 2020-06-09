# https://github.com/jupyter/docker-stacks
# https://github.com/jupyter-incubator/sparkmagic
# https://github.com/jupyter/docker-stacks/tree/master/scipy-notebook

#ARG BASE_CONTAINER=jupyter/scipy-notebook:d4cbf2f80a2a
ARG BASE_CONTAINER=cyliu0204/jupyter:scipy_nb_gpu
FROM $BASE_CONTAINER

LABEL maintainer="Aliaksandr Sasnouskikh <jaahstreetlove@gmail.com>"

# Install Sparkmagic kernel
# https://github.com/jupyter-incubator/sparkmagic/blob/master/Dockerfile.jupyter
USER root

# This is needed because requests-kerberos fails to install on debian due to missing linux headers
RUN conda install requests-kerberos -y

USER $NB_UID

RUN pip install --upgrade pip && \
    pip install --upgrade --ignore-installed setuptools && \
    pip install sparkmagic==0.15.0

RUN mkdir -p /home/$NB_USER/.sparkmagic
COPY sparkmagic/example_config.json /home/$NB_USER/.sparkmagic/config.json

RUN jupyter nbextension enable --py --sys-prefix widgetsnbextension && \
    jupyter-kernelspec install --user $(pip show sparkmagic | grep Location | cut -d" " -f2)/sparkmagic/kernels/sparkkernel && \
    jupyter-kernelspec install --user $(pip show sparkmagic | grep Location | cut -d" " -f2)/sparkmagic/kernels/pysparkkernel && \
    jupyter serverextension enable --py sparkmagic


RUN pip install dask_labextension && \
    jupyter labextension install dask-labextension && \
    jupyter serverextension enable dask_labextension

RUN pip install --upgrade jupyterlab-git && \
    jupyter lab build 

RUN     pip install jupyterlab-s3-browser && \
    jupyter labextension install jupyterlab-s3-browser && \
    jupyter serverextension enable --py jupyterlab_s3_browser


USER root
RUN chown $NB_USER /home/$NB_USER/.sparkmagic/config.json

# Misc
RUN mkdir -p /home/$NB_USER/notebooks && \
    chmod -R 777 /home/$NB_USER/notebooks

COPY entrypoint.sh /opt/
COPY singleuser-entrypoint.sh /opt/

ENTRYPOINT ["tini", "-g", "--"]
CMD [ "/opt/entrypoint.sh" ]

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID

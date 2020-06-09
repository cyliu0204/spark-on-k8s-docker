FROM  cyliu0204/jupyter:4.4.0-sparkmagic_0.15.0

USER $NB_USER
RUN conda install pytorch torchvision -c pytorch
RUN conda clean -a -f -y && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

WORKDIR /home/$NB_USER

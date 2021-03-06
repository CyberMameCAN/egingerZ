ARG BASE_IMAGE=python:3.8-slim-buster                                                                                
FROM $BASE_IMAGE                                                                                                     
                                                                                                                     
# system update & package install                                                                                    
RUN apt-get -y update && \                                                                                           
    apt-get install -y --no-install-recommends \                                                                     
    build-essential \                                                                                                
    openssl libssl-dev \                                                                                             
    && apt-get clean \                                                                                               
    && rm -rf /var/lib/apt/lists/*                                                                                   
                                                                                                                     
# Add Tini                                                                                                           
ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

#Add general user account
ARG USER_NAME=app
ARG USER_UID=1000
ARG PASSWD=password

RUN useradd -m -s /bin/bash -u $USER_UID $USER_NAME && \
    gpasswd -a $USER_NAME sudo && \
    echo "${USER_NAME}:${PASSWD}" | chpasswd && \
    echo "${USER_NAME} ALL=(ALL) ALL" >> /etc/sudoers

#COPY FastAPI source code etc. into the container
COPY ./app /app/app
COPY ./pyproject.toml /app/pyproject.toml
#Poetry used in local environment.Add the lock file, if any
# COPY ./poetry.lock /app/poetry.lock
RUN chown -R ${USER_NAME}:${USER_NAME} /app


USER $USER_NAME
WORKDIR /app
# pip & poetry
ENV PATH $PATH:/home/${USER_NAME}/.local/bin
RUN python3 -m pip install --user --upgrade pip && \
    python3 -m pip install poetry --user && \
    poetry config virtualenvs.in-project true && \
    poetry install && \
    rm -rf ~/.cache/pip/* && \
    rm -rf ~/.cache/pypoetry/*

#Put the PATH in the virtual environment created by Poetry (set the poetry config in advance to make the PATH of the virtual environment unique)
ENV PATH=/app/.venv/bin:$PATH

# Configration
EXPOSE 8000

# Execute
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0"]
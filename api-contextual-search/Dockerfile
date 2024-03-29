# base image derivation 
FROM python:3.7-slim-stretch

# initial arg env setup  
ARG NGT_ARG_VERSION="1.13.7"
ENV NGT_ENV_VERSION=${NGT_ARG_VERSION}

ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Europe/Paris

# initial config and setup  
RUN apt-get update --fix-missing &&\
    apt-get install --yes --no-install-recommends \ 
        tzdata dialog apt-utils \ 
        git gcc curl pkg-config build-essential cmake wget unzip \ 
        ffmpeg libsm6 libxext6 libpcre3 libpcre3-dev 

# user creation && pull source  
RUN useradd --gid root --create-home solver 
WORKDIR /home/solver

# virtual environment setup and activation  
ENV VIRTUAL_ENV=/opt/venv 
RUN chmod -R g+rwx /home/solver && \ 
    python -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

# install packages 
RUN pip install --upgrade pip setuptools wheel

# install ngt from source 
RUN wget https://github.com/yahoojapan/NGT/archive/refs/tags/v${NGT_ENV_VERSION}.zip && \
    unzip v${NGT_ENV_VERSION} && \
    cd NGT-${NGT_ENV_VERSION} && \ 
    mkdir build && \ 
    cd build && \ 
    cmake -DNGT_SHARED_MEMORY_ALLOCATOR=ON .. && \
    make && \ 
    make install && \
    ldconfig /usr/local/lib

# install python ngt binding 
RUN pip install --upgrade pip setuptools wheel numpy pybind11 pyzmq loguru click opencv-python tqdm scikit-learn scipy torchvision && \
    cd NGT-${NGT_ENV_VERSION}/python &&\
    python setup.py sdist && \ 
    pip install dist/ngt-${NGT_ENV_VERSION}.tar.gz

# copy source code from local to docker environment
COPY . ./ 

# define environment variables for the running process  
ENV INDEX='index/'
ENV DISTANCE_TYPE='Cosine'
ENV DIMENSION=512
ENV BATCH_SIZE=5000
ENV SERVER_PORT=8800
ENV FEATURES_LOCATION='location/'

# expose zmq server port 
EXPOSE $SERVER_PORT

# entrypoint
CMD ["python", "server.py"]


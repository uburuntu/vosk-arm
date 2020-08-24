FROM python:3.7-buster

# Pass a required ARM architecture while building an image, e.g.
# docker build --build-arg ARCH=ARMV8 -t alphacep/kaldi-vosk-server:armv8 .
ARG ARCH=ARM64
ENV RUVERSION 0.10

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        g++ \
        gfortran \
        bzip2 \
        unzip \
        make \
        wget \
        git \
        zlib1g-dev \
        patch \
        ca-certificates \
        swig \
        cmake \
        xz-utils \
    && rm -rf /var/lib/apt/lists/*

RUN \
    git clone -b lookahead --single-branch https://github.com/alphacep/kaldi /opt/kaldi \
    && cd /opt/kaldi/tools \
    && sed -i 's:status=0:exit 0:g' extras/check_dependencies.sh \
    && sed -i 's:openfst_add_CXXFLAGS = -g -O2:openfst_add_CXXFLAGS = -g -O3:g' Makefile \
    && sed -i 's:--enable-ngram-fsts:--enable-ngram-fsts --disable-bin:g' Makefile \
    && sed -i 's:python:python3:g' extras/install_openblas.sh \
    && sed -i 's:USE_LOCKING=1:TARGET=${ARCH} USE_LOCKING=1:g' extras/install_openblas.sh \
    && make -j $(nproc) openfst cub \
    && extras/install_openblas.sh \
    && cd /opt/kaldi/src \
    && ./configure --mathlib=OPENBLAS --shared \
    && sed -i 's: -O1 : -O3 :g' kaldi.mk \
    && make HOSTCC=gcc TARGET=${ARCH} -j $(nproc) online2 lm

RUN \
    git clone https://github.com/alphacep/vosk-api /opt/vosk-api \
    && sed -i 's:--max-active=3000:--max-active=7000:g' /opt/vosk-api/src/model.cc \
    && sed -i 's:--beam=10.0:--beam=13.0:g' /opt/vosk-api/src/model.cc \
    && sed -i 's:--lattice-beam=2.0:--lattice-beam=6.0:g' /opt/vosk-api/src/model.cc \
    && cd /opt/vosk-api/python \
    && KALDI_ROOT=/opt/kaldi python3 ./setup.py install --user --single-version-externally-managed --root=/ \
    && git clone https://github.com/alphacep/vosk-server /opt/vosk-server \
    && rm -rf /opt/kaldi

RUN mkdir /opt/vosk-model-ru \
   && cd /opt/vosk-model-ru \
   && wget -q http://alphacephei.com/kaldi/models/vosk-model-ru-${RUVERSION}.zip \
   && unzip vosk-model-ru-${RUVERSION}.zip \
   && mv vosk-model-ru-${RUVERSION} model \
   && rm -rf model/extra \
   && rm -rf model/rnnlm \
   && rm -rf vosk-model-ru-${RUVERSION}.zip

WORKDIR /opt/vosk-server/websocket
CMD [ "python3", "./asr_server.py", "/opt/vosk-model-ru/model" ]

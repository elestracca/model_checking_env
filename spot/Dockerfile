FROM python:3.11-slim


RUN apt update && apt install -y curl build-essential libpython3-dev


RUN curl -L https://www.lrde.epita.fr/dload/spot/spot-2.11.6.tar.gz | tar xz -C /opt/

RUN mkdir -p /opt/spot-2.11.6/install
RUN cd /opt/spot-2.11.6 && \
    ./configure \
    --enable-python \
    --prefix=/opt/spot-2.11.6/install \
    && make -j6 && \
    make install && \
    ldconfig


ENV PYTHONPATH=/opt/spot-2.11.6/install/lib/python3.11/site-packages:${PYTHONPATH}

ENV LD_LIBRARY_PATH=/opt/spot-2.11.6/lib:${LD_LIBRARY_PATH}

ENV PATH=/opt/spot-2.11.6/bin:${PATH}

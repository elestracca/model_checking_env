FROM spot_image:latest AS spot 

#FROM storm_image:latest
FROM test_ubuntu:latest


COPY --from=spot /opt/spot-2.11.6 /opt/spot-2.11.6

RUN find /opt/spot-2.11.6/install/lib/python3.11/site-packages/ -type f -name "*.cpython-311-x86_64-linux-gnu.*" \
    -exec bash -c 'for f; do mv "$f" "${f/cpython-311/cpython-313}"; done' bash {} +
# COPY --from=spot /opt/spot-2.11.6/install/python3.11/site-packages /opt/venv/lib/python3.12/site-packages
ENV PYTHONPATH=/opt/spot-2.11.6/install/lib/python3.11/site-packages:${PYTHONPATH}

ENV LD_LIBRARY_PATH=/opt/spot-2.11.6/lib:${LD_LIBRARY_PATH}

ENV PATH=/opt/spot-2.11.6/bin:${PATH}
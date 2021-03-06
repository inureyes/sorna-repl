FROM lablup/kernel-base:latest

# Install Python
# reference: https://github.com/docker-library/python/blob/master/3.6/alpine/Dockerfile
ENV PATH=/usr/local/bin:$PATH
ENV LANG=C.UTF-8

ENV PYTHONUNBUFFERED 1
ENV PYTHON_VERSION 2.7.13
ENV PYTHON_PIP_VERSION 9.0.1

ENV GPG_KEY C01E1CAD5EA2C4F0B8E3571504C367C218ADD4FF

RUN set -ex \
        && apk add --no-cache --virtual .fetch-deps \
                gnupg \
                openssl \
                tar \
                xz \
        \
        && wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
        && wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
        && export GNUPGHOME="$(mktemp -d)" \
        && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
        && gpg --batch --verify python.tar.xz.asc python.tar.xz \
        && rm -r "$GNUPGHOME" python.tar.xz.asc \
        && mkdir -p /usr/src/python \
        && tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
        && rm python.tar.xz \
        \
        && apk add --no-cache --virtual .build-deps  \
                bzip2-dev \
                gcc \
                gdbm-dev \
                libc-dev \
                linux-headers \
                make \
                ncurses-dev \
                openssl \
                openssl-dev \
                pax-utils \
                readline-dev \
                sqlite-dev \
                tcl-dev \
                tk \
                tk-dev \
                xz-dev \
                zlib-dev \
# add build deps before removing fetch deps in case there's overlap
        && apk del .fetch-deps \
        \
        && cd /usr/src/python \
        && ./configure \
                --enable-loadable-sqlite-extensions \
                --enable-shared \
        && make -j$(getconf _NPROCESSORS_ONLN) \
        && make install \
        \
                && wget -O /tmp/get-pip.py 'https://bootstrap.pypa.io/get-pip.py' \
                && python2 /tmp/get-pip.py "pip==$PYTHON_PIP_VERSION" \
                && rm /tmp/get-pip.py \
# we use "--force-reinstall" for the case where the version of pip we're trying to install is the same as the version bundled with Python
# ("Requirement already up-to-date: pip==8.1.2 in /usr/local/lib/python3.6/site-packages")
# https://github.com/docker-library/python/pull/143#issuecomment-241032683
        && pip install --no-cache-dir --upgrade --force-reinstall "pip==$PYTHON_PIP_VERSION" \
# then we use "pip list" to ensure we don't have more than one pip version installed
# https://github.com/docker-library/python/pull/100
        && [ "$(pip list |tac|tac| awk -F '[ ()]+' '$1 == "pip" { print $2; exit }')" = "$PYTHON_PIP_VERSION" ] \
        \
        && find /usr/local -depth \
                \( \
                        \( -type d -a -name test -o -name tests \) \
                        -o \
                        \( -type f -a -name '*.pyc' -o -name '*.pyo' \) \
                \) -exec rm -rf '{}' + \
        && runDeps="$( \
                scanelf --needed --nobanner --recursive /usr/local \
                        | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
                        | sort -u \
                        | xargs -r apk info --installed \
                        | sort -u \
        )" \
        && apk add --virtual .python-rundeps $runDeps \
        && apk del .build-deps \
        && rm -rf /usr/src/python ~/.cache

# Lablup extension: install Sorna's common Python packages
RUN apk add --no-cache --virtual .pybuild-deps gcc g++ make libc-dev \
    && apk add --no-cache --virtual .sorna-deps zeromq libuv \
    && pip install --no-cache-dir pyzmq namedlist simplejson six "python-dateutil<2" \
    && apk del .pybuild-deps

# vim: sts=4 sw=4 et

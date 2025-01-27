# Software Name: its-client
# SPDX-FileCopyrightText: Copyright (c) 2016-2022 Orange
# SPDX-License-Identifier: MIT License
#
# This software is distributed under the MIT license, see LICENSE.txt.txt file for more details.
#
# Author: Frédéric GARDES <frederic.gardes@orange.com> et al.
# Software description: This Intelligent Transportation Systems (ITS) [MQTT](https://mqtt.org/) client based on the [JSon](https://www.json.org) [ETSI](https://www.etsi.org/committee/its) specification transcription provides a ready to connect project for the mobility (connected and autonomous vehicles, road side units, vulnerable road users,...).
FROM python:3.8-buster as builder

LABEL description="Python builder to compile"
LABEL maintainer="frederic.gardes@orange.com"
LABEL authors=["frederic.gardes@orange.com"]
LABEL url="https://github.com/Orange-OpenSource/its-client"

RUN python --version

WORKDIR /usr/src/app

COPY its_client its_client
COPY requirements.txt .
COPY setup.cfg .
COPY setup.py .

RUN pip install \
        --disable-pip-version-check \
        -r requirements.txt

RUN python -m unittest && \
    python setup.py test && \
    python setup.py bdist_wheel

FROM python:3.8-buster

LABEL description="Python ITS client"
LABEL maintainer="frederic.gardes@orange.com"
LABEL authors=["frederic.gardes@orange.com"]
LABEL url="https://github.com/Orange-OpenSource/its-client"

# Install curl for healthcheck
RUN apt-get update && \
    apt-get install --no-install-recommends -y \
        curl=7.* \
        procps=2:* && \
    rm -rf /var/lib/apt/lists/*

# Create app directory
WORKDIR /usr/src/app

RUN groupadd -r itsclient && useradd --no-log-init -r -g itsclient itsclient && \
    mkdir /usr/src/app/log && \
    printf "Version 1.0\n" > /usr/src/app/log/RELEASE

COPY --from=builder --chown=itsclient:itsclient /usr/src/app/dist/its_client-1.0.0-py3-none-any.whl .

COPY docker-entrypoint.sh .

RUN chown -R itsclient:itsclient /usr/src/app && \
    chmod -R ug+rwx /usr/src/app && \
    pip install its_client-1.0.0-py3-none-any.whl

# no parameters today, just to check de lint
CMD [""]

RUN mkdir /var/log/its_client &&\
    chown itsclient:itsclient /var/log/its_client
USER itsclient

ENTRYPOINT ["/usr/src/app/docker-entrypoint.sh"]

HEALTHCHECK CMD python PYTHONPATH='$PYTHONPATH:/usr/local/lib/python3.8/site-packages/its_client' python /usr/local/lib/python3.8/site-packages/its_client/main.py --help || exit 1

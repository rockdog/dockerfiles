FROM debian:stretch-slim

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    python2.7 \
    python2.7-dev \
    python-setuptools \
    libxml2-dev \
    libxslt-dev \
    libxml2 \
    libxslt1.1 \
    zlib1g-dev \
    ; \
    rm -rf /var/lib/apt/lists/*

RUN useradd -m -d /home/app -k /etc/skel -s /bin/bash app

USER app

RUN set -eux; \
    easy_install --user pip; \
    /home/app/.local/bin/pip install virtualenv==16.7.9; \
    /home/app/.local/bin/virtualenv /home/app/pootle/venv

WORKDIR /home/app/pootle

RUN mkdir db log translations exports

RUN venv/bin/pip install --upgrade pip setuptools
RUN venv/bin/pip install Pootle==2.8.2 django-tastypie==0.14.3

COPY pootle.conf pootle.conf

RUN venv/bin/pootle collectstatic --noinput --clear --config pootle.conf
RUN venv/bin/pootle assets build --config pootle.conf

# EXPOSE 8000

CMD ["venv/bin/pootle", "runserver", "--insecure", "--config=pootle.conf"]
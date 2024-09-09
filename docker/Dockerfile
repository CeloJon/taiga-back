# Use a Red Hat Universal Base Image (UBI) como base
FROM registry.access.redhat.com/ubi8/ubi:latest
LABEL maintainer="support@taiga.io"

# Set environment variables to avoid interactive prompts
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONFAULTHANDLER=1

# Install system dependencies
RUN dnf -y install \
    python3 \
    python3-pip \
    python3-virtualenv \
    git \
    gettext \
    libpq \
    libpq-devel \
    net-tools \
    procps-ng \
    wget \
    gnupg2 \
    && dnf clean all

# Create and activate a virtual environment
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Get the code
COPY . /taiga-back
WORKDIR /taiga-back

# Install Python dependencies
RUN python -m pip install --upgrade pip \
    && python -m pip install --no-cache-dir wheel \
    && python -m pip install --no-cache-dir -r requirements.txt \
    && python -m pip install --no-cache-dir -r requirements-contribs.txt \
    && python manage.py compilemessages \
    && python manage.py collectstatic --no-input \
    && chmod +x docker/entrypoint.sh \
    && chmod +x docker/async_entrypoint.sh \
    && cp docker/config.py settings/config.py \
    && groupadd --system taiga --gid=999 \
    && useradd --system --no-create-home --gid taiga --uid=999 --shell=/bin/bash taiga \
    && mkdir -p /taiga-back/media/exports \
    && chown -R taiga:taiga /taiga-back

# Clean up
RUN dnf remove -y \
    git \
    net-tools \
    procps-ng \
    wget \
    && dnf autoremove -y \
    && rm -rf /var/cache/dnf \
    && rm -rf /root/.cache \
    && rm requirements.txt \
    && rm requirements-contribs.txt \
    && find . -name '__pycache__' -exec rm -r '{}' + \
    && find . -name '*pyc' -exec rm -r '{}' + \
    && find . -name '*po' -exec rm -r '{}' +

ENV DJANGO_SETTINGS_MODULE=settings.config

EXPOSE 8000
ENTRYPOINT ["./docker/entrypoint.sh"]

FROM python:3.11-slim

LABEL maintainer="support@taiga.io"

# Configurações do sistema
RUN yum update -y && \
    yum install -y \
    gcc \
    cairo-devel \
    libffi-devel \
    gdk-pixbuf2 \
    pango \
    pango-devel \
    cairo-gobject \
    cairo-gobject-devel \
    && yum clean all

# Configurações Python
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONFAULTHANDLER=1

# Use a virtualenv
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copie o código
COPY . /taiga-back
WORKDIR /taiga-back

# Instale dependências Python
RUN python -m pip install --upgrade pip && \
    python -m pip install --no-cache-dir wheel && \
    python -m pip install --no-cache-dir -r requirements.txt && \
    python -m pip install --no-cache-dir -r requirements-contribs.txt && \
    python manage.py compilemessages && \
    python manage.py collectstatic --no-input && \
    chmod +x docker/entrypoint.sh && \
    chmod +x docker/async_entrypoint.sh && \
    cp docker/config.py settings/config.py && \
    groupadd --system taiga --gid=999 && \
    useradd --system --no-create-home --gid taiga --uid=999 --shell=/bin/bash taiga && \
    mkdir -p /taiga-back/media/exports && \
    chown -R taiga:taiga /taiga-back

EXPOSE 8000
ENTRYPOINT ["./docker/entrypoint.sh"]

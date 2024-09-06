# Usar a imagem base UBI8 do Red Hat com Python 3.11
FROM registry.access.redhat.com/ubi8/python-3.11

USER root

ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONFAULTHANDLER=1

# Usar um virtualenv
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Obter o código
COPY . /taiga-back
WORKDIR /taiga-back

# Instalar dependências do sistema
RUN set -eux; \
    dnf install -y \
       gcc \
       gettext \
       libpq-devel \
       git \
       net-tools \
       procps-ng \
       wget \
       gnupg2; \
    # Instalar gosu
    GOSU_VERSION=1.12; \
    ARCH=$(uname -m); \
    wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$ARCH"; \
    wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$ARCH.asc"; \
    chmod +x /usr/local/bin/gosu; \
    # Verificar assinatura do gosu
    export GNUPGHOME="$(mktemp -d)"; \
    gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4; \
    gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu; \
    command -v gpgconf && gpgconf --kill all || :; \
    rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc; \
    # Instalar dependências do Taiga
    python -m pip install --upgrade pip; \
    python -m pip install wheel; \
    python -m pip install -r requirements.txt; \
    python -m pip install -r requirements-contribs.txt; \
    python manage.py compilemessages; \
    python manage.py collectstatic --no-input; \
    chmod +x docker/entrypoint.sh; \
    chmod +x docker/async_entrypoint.sh; \
    cp docker/config.py settings/config.py; \
    # Criar grupo e usuário taiga para usar e dar permissões sobre o código
    groupadd --system taiga --gid=999; \
    useradd --system --no-create-home --gid taiga --uid=999 --shell=/bin/bash taiga; \
    mkdir -p /taiga-back/media/exports; \
    chown -R taiga:taiga /taiga-back; \
    # Remover arquivos e pacotes desnecessários
    dnf remove -y \
       gcc \
       gettext \
       git \
       libpq-devel \
       net-tools \
       procps-ng \
       wget; \
    dnf autoremove -y; \
    rm -rf /var/cache/dnf; \
    rm -rf /root/.cache; \
    # Limpar taiga
    rm requirements.txt; \
    rm requirements-contribs.txt; \
    find . -name '__pycache__' -exec rm -r '{}' +; \
    find . -name '*pyc' -exec rm -r '{}' +; \
    find . -name '*po' -exec rm -r '{}' +

ENV DJANGO_SETTINGS_MODULE=settings.config

EXPOSE 8000
ENTRYPOINT ["./docker/entrypoint.sh"]

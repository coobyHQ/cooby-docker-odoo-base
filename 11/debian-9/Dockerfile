FROM bitnami/minideb-extras:stretch-r229
LABEL maintainer "Cooby <admin@cooby.io>"

# Install required system packages and dependencies
RUN install_packages ghostscript imagemagick libbsd0 libbz2-1.0 libc6 libedit2 libffi6 libgcc1 libgcrypt20 libgmp10 libgnutls30 libgpg-error0 libhogweed4 libicu57 libidn11 libldap-2.4-2 liblzma5 libncurses5 libnettle6 libp11-kit0 libpq5 libreadline7 libsasl2-2 libsqlite3-0 libssl1.1 libstdc++6 libtasn1-6 libtinfo5 libxml2 libxslt1.1 zlib1g
# RUN bitnami-pkg install python-3.6.7-20 --checksum 63b9373c246fde6435aaf3b35ae06723ff8fd035304190c7fb3b3378744650b1
RUN bitnami-pkg install postgresql-client-10.6.0-20 --checksum 566509fb3c817cae6bab80d678fe3ee991d04ef508cb7fae552aa4f811373da6
# RUN bitnami-pkg install node-7.10.1-1 --checksum 53362288e961d298a724353710a31901d4cb8237894a26a2f676b35450c02975
# RUN bitnami-pkg unpack odoo-11.0.20181215-0 --checksum 8d44b825c952d38f472fccb51a9fa9966aabe0c32e582cc71ab6f52b5b576add

COPY rootfs /
ENV COOBY_APP_NAME="odoo-base" \
    COOBY_IMAGE_VERSION="11.0.20181215-debian-9-r2" \
    POSTGRESQL_HOST="postgresql" \
    POSTGRESQL_PASSWORD="" \
    POSTGRESQL_PORT_NUMBER="5432" \
    POSTGRESQL_USER="postgres" \
    SMTP_HOST="" \
    SMTP_PASSWORD="" \
    SMTP_PORT="" \
    SMTP_PROTOCOL="" \
    SMTP_USER=""

EXPOSE 8069 8071

ENTRYPOINT [ "/app-entrypoint.sh" ]
CMD [ "nami", "start", "--foreground", "odoo-base" ]

FROM ubuntu:14.04
MAINTAINER João Paulo de Souza <vibedigital@gmail.com>

# generate locales
RUN  locale-gen pt_BR; locale-gen pt_BR.UTF-8; sudo dpkg-reconfigure locales; sudo update-locale LANG=pt_BR.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends \
            ca-certificates \
            curl \
            python-pip \
            nodejs \
            npm \
            python-support \
            python-pyinotify \
	    python-openssl \
        && npm install -g less less-plugin-clean-css \
        && ln -s /usr/bin/nodejs /usr/bin/node \
        && curl -o wkhtmltox.deb -SL http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linux-jessie-amd64.deb \
        && echo '40e8b906de658a2221b15e4e8cd82565a47d7ee8 wkhtmltox.deb' | sha1sum -c - \
        && dpkg --force-depends -i wkhtmltox.deb \
        && apt-get -y install -f --no-install-recommends \
        && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false npm \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# Install Odoo
ENV ODOO_VERSION 8.0
ENV ODOO_RELEASE 20150401
RUN set -x; \
        curl -o odoo.deb -SL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}.${ODOO_RELEASE}_all.deb \
        && dpkg --force-depends -i odoo.deb \
        && apt-get update \
        && apt-get -y install -f --no-install-recommends \
        && rm -rf /var/lib/apt/lists/* odoo.deb

#instal brazilian localization dependencies
RUN set -x; \
    apt-get update && \
	apt-get -yq install \
	python-dev \ 
	libxmlsec1-dev \
	libxml2-dev \
	&& pip install pyxmlsec --allow-unverified pyxmlsec \
		lxml \
		pySPED
#install brazilian localization
RUN apt-get update && \
	  apt-get -yq install git
	 
RUN git clone -b 8.0 https://github.com/JoaoPauloSouza/l10n-brazil.git  /mnt/extra-addons/l10n-brazil 
RUN git clone -b 8.0 https://github.com/JoaoPauloSouza/odoo-brazil-eletronic-documents.git /mnt/extra-addons/odoo-brazil-eletronic-documents
RUN git clone -b 8.0 https://github.com/JoaoPauloSouza/account-fiscal-rule.git /mnt/extra-addons/account-fiscal-rule
RUN git clone -b 8.0 https://github.com/OCA/server-tools.git /mnt/extra-addons/server-tools 


# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./openerp-server.conf /etc/odoo/
RUN chown odoo /etc/odoo/openerp-server.conf

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV OPENERP_SERVER /etc/odoo/openerp-server.conf

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["openerp-server"]

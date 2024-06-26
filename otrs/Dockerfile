#OTRS ticketing system docker image.
FROM centos:7
LABEL maintainer="Juan Luis Baptiste <juan@juanbaptiste.tech>"
ENV OTRS_VERSION=6.5.8-01
ENV OTRS_ROOT "/opt/otrs/"
ENV OTRS_BACKUP_DIR "/var/otrs/backups"
ENV OTRS_CONFIG_DIR "${OTRS_ROOT}Kernel"
ENV OTRS_CONFIG_MOUNT_DIR "/config/"
ENV OTRS_SKINS_MOUNT_DIR "/skins/"
ENV SKINS_PATH "${OTRS_ROOT}/var/httpd/htdocs/skins/"
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8

RUN yum install -y yum-plugin-fastestmirror && \
    yum install -y epel-release && \
    yum -y install https://harbottle.gitlab.io/epmel/7/x86_64/epmel-release.rpm && \
    yum update -y && \
    yum -y install bzip2 cronie httpd mysql mod_perl perl-Apache-DBI \
    perl-core "perl(Crypt::SSLeay)" "perl(Net::LDAP)" "perl(URI)" \
    procmail "perl(Date::Format)" "perl(LWP::UserAgent)" "perl(Data::UUID)" \
    "perl(Net::DNS)" "perl(IO::Socket::SSL)" "perl(XML::Parser)" \
    "perl(Apache3::Reload)" "perl(Crypt::Eksblowfish::Bcrypt)" \
    "perl(Encode::HanExtra)" "perl(GD)" "perl(GD::Text)" "perl(GD::Graph)" \
    "perl(JSON::XS)" "perl(Mail::IMAPClient)" "perl(PDF::API2)" "perl(DateTime)" \
    "perl(Text::CSV_XS)" "perl(YAML::XS)" "perl(Text::CSV_XS)" "perl(DBD::mysql)" \
    perl-Moo "perl(namespace::clean)" "perl(Crypt::Random::Source)" "perl(Exporter::Tiny)" \
    "perl(Math::Random::ISAAC)" "perl(Math::Random::Secure)" "perl(Module::Find)" \
    "perl(Types::TypeTiny)" rsyslog supervisor tar which cpanminus && \
    cpanm iCal::Parser && \
    yum install -y https://download.znuny.org/releases/RPMS/rhel/7/znuny-${OTRS_VERSION}.noarch.rpm && \
    /opt/otrs/bin/otrs.CheckModules.pl && \
    yum clean all
# Add scripts and function files
COPY *.sh /
#Supervisord configuration
COPY etc/supervisord.d/otrs.ini /etc/supervisord.d/
RUN chmod 755 /*.sh  && \
    cp ${OTRS_ROOT}/var/httpd/htdocs/index.html /var/www/html && \
    chmod 644 /var/www/html/index.html && \
    sed -i 's/\bindex.html\b/& index.pl/' /etc/httpd/conf/httpd.conf && \
    echo "+ : otrs : cron crond" |cat >> /etc/security/access.conf                && \
    sed -i -e '/pam_loginuid.so/ s/^#*/#/' /etc/pam.d/crond                       && \
    sed -i -e "s/^nodaemon=false/nodaemon=true/" /etc/supervisord.conf && \
    cat /etc/supervisord.d/otrs.ini >> etc/supervisord.conf && \
    sed -i -e '/<ValidateModule>Kernel::System::SysConfig::StateValidate<\/ValidateModule>/ s/^#*/#/' \
        ${OTRS_ROOT}Kernel/Config/Files/XML/Ticket.xml  && \
    mkdir -p ${OTRS_ROOT}var/{run,tmp}/ && \
    # perl -cw ${OTRS_ROOT}bin/cgi-bin/index.pl && \
    # perl -cw ${OTRS_ROOT}bin/cgi-bin/customer.pl && \
    # perl -cw ${OTRS_ROOT}bin/otrs.Console.pl && \
    sed -i -e '/\$ModLoad imjournal/ s/^#*/#/' /etc/rsyslog.conf && \
    sed -i -e '/\$IMJournalStateFile imjournal.state/ s/^#*/#/' /etc/rsyslog.conf && \
    sed -i 's/\(^\$OmitLocalLogging \).*/\1off/' /etc/rsyslog.conf && \
    rm /etc/rsyslog.d/listen.conf
#To be able to use a host-mounted volume for OTRS configuration we need to move
#away the contents of ${OTRS_CONFIG_DIR} to another place and move them back
#on first container run (see check_host_mount_dir on functions.sh), after the
#host-volume is mounted. The same for the skins.
RUN mv ${OTRS_CONFIG_DIR} / && \
    mv ${SKINS_PATH} / && \
    touch ${OTRS_ROOT}var/tmp/firsttime && \
    ln -sf /dev/stdout /var/log/httpd/access_log && \
    ln -sf /dev/stderr /var/log/httpd/error_log && \
    ln -sf /dev/stdout /var/log/cron
EXPOSE 80
CMD ["/run.sh"]

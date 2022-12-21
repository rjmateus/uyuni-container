FROM registry.suse.com/bci/bci-init:15.4

RUN zypper ar -G http://download.opensuse.org/update/leap/15.4/backports/ backports_update_repo
RUN zypper ar -G http://download.opensuse.org/distribution/leap/15.4/repo/oss/ os_pool_repo
RUN zypper ar -G http://download.opensuse.org/update/leap/15.4/oss/ os_update_repo
RUN zypper ar -G http://download.opensuse.org/repositories/systemsmanagement:/Uyuni:/Master/images/repo/Uyuni-Server-POOL-x86_64-Media1/ server_pool_repo
RUN zypper ar -G http://download.opensuse.org/update/leap/15.4/sle/ sle_update_repo
RUN zypper ar -G http://download.opensuse.org/repositories/systemsmanagement:/Uyuni:/Master/images/repo/Testing-Overlay-POOL-x86_64-Media1/ testing_overlay_devel_repo

copy setup_env.sh /root/setup_env.sh

RUN zypper ref
RUN zypper -n in vi
RUN zypper -n in python3-simplejson
RUN zypper --non-interactive install --auto-agree-with-licenses --force-resolution patterns-uyuni_server
RUN zypper mr -d repo-backports-update repo-sle-update

RUN echo 'root:root123!' | chpasswd
# hack to correct the report db script to work on containers
RUN cp /usr/bin/uyuni-setup-reportdb /usr/bin/uyuni-setup-reportdb.original
RUN sed -i 's/sysctl kernel.shmmax/#sysctl kernel.shmmax/g' /usr/bin/uyuni-setup-reportdb
RUN sed -i 's/-a "$PG_SOCKET"//g' /usr/bin/uyuni-setup-reportdb


EXPOSE 443/tcp
EXPOSE 80/tcp
EXPOSE 4505/tcp
EXPOSE 4506/tcp


VOLUME [ \
# DATA
"/var/lib/pgsql", "/var/spacewalk", "/var/cache", "/etc/salt", \
# salt
"/usr/share/susemanager/", \
"/usr/share/salt-formulas/", \
"/srv/susemanager/salt", \
"/srv/salt",  \
# home(s)
"/root/", \
"/home/", \
# Logs
"/var/log/rhn/", \
# configurations
"/etc/rhn", "/etc/pki/", "/etc/apache2/", "/etc/tomcat/"]

CMD ["/usr/lib/systemd/systemd"]
HEALTHCHECK --interval=5s --timeout=5s --retries=5 CMD ["/usr/bin/systemctl", "is-active", "multi-user.target"]

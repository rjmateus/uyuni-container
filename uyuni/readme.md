Instructions on how to build and ocnfigure uyuni container image. Tested and documented with docker and podman.

# Docker - build and run

## Build with docker

```
docker build --label suma -t uyuni/server:1.0 --add-host=download.opensuse.org:195.135.221.134 .
```

## Run with docker

One should use a host name that can be resolved to the machine where the container is starting.

```
docker run --tmpfs /run -v /sys/fs/cgroup:/sys/fs/cgroup:rw --cgroupns=host --add-host=download.opensuse.org:195.135.221.134 -h "<HOST-NAME>" -p 443:443 -p 80:80 -p 4505:4505 -p 4506:4506 -it uyuni/server:1.0
```

# Podman - build and run

## Build with Podman

```
podman build --label suma -t uyuni/server:1.0 --add-host=download.opensuse.org:195.135.221.134 .
```

## Run with Podman

```
podman run -ti --tmpfs /run -v /sys/fs/cgroup:/sys/fs/cgroup:rw --cgroupns=host --add-host=download.opensuse.org:195.135.221.134 -h "uyuni-container-srv.tf.local" \
-p 443:443 -p 80:80 -p 4505:4505 -p 4506:4506  \
uyuni/server:1.0
```
## Run with Volumes

A simple way to manage storage requirements is to mount necessary storage on the container host to '/var/lib/containers/storage/volumes'.  By using this method, the mapped podman volumes enumerated below will appear there.  For eample, creating an LVM LV or NFS mount there with adequate capacity will persist and prevent filling up the '/' filesystem on the container host.

To run the container. edit the '-h' portion of this command with the desired FQDN of the containerized SUMA/Uyuni server.

```
podman run -ti --tmpfs /run -v /sys/fs/cgroup:/sys/fs/cgroup:rw --cgroupns=host --add-host=download.opensuse.org:195.135.221.134 -h "uyuni-container-srv.tf.local" \
--name=uyuni-server \
-p 443:443 -p 80:80 -p 4505:4505 -p 4506:4506  \
-v pgsql:/var/lib/pgsql \
-v spacewalk:/var/spacewalk \
-v cache:/var/cache \
-v share-susemanager:/usr/share/susemanager \
-v share-salt-formulas:/usr/share/salt-formulas \
-v srv-susemanager_salt:/srv/susemanager/salt \
-v srv-salt:/srv/salt \
-v srv-www:/srv/www/ \
-v etc-salt:/etc/salt \
-v etc_rhn:/etc/rhn \
-v var_log_rhn:/var/log/rhn \
-v pki_tls_certs:/etc/pki \
-v apache2:/etc/apache2 \
-v root:/root \
-v tomcat:/etc/tomcat/ \
localhost/uyuni/server:1.0
```
## Local bindings

TODO

Bind to local folders, needs to be a special since no default data is copied (?) and owners can be different.
Can be usefull id user want to start it using a previous rpm base deployment folders.
Should also consider if we should move this to podman volumes.

run the container
mkdir -p var/lib/pgsql
mkdir -p var/spacewalk && \
mkdir -p var/cache && \
mkdir -p usr/share/susemanager && \
mkdir -p usr/share/salt-formulas/ && \
mkdir -p srv/susemanager/salt && \
mkdir -p srv/salt && \
mkdir -p etc/salt && \
mkdir -p etc/rhn && \
mkdir -p var/log/rhn/
```
podman run -ti --tmpfs /run -v /sys/fs/cgroup:/sys/fs/cgroup:rw --cgroupns=host --add-host=download.opensuse.org:195.135.221.134 -h "m43-minion-suse.tf.local" \
--name=uyuni-server \
-p 443:443 -p 80:80 -p 4505:4505 -p 4506:4506  \
-v /var/lib/pgsql:/var/lib/pgsql \
-v /var/spacewalk:/var/spacewalk \
-v /var/cache:/var/cache \
-v /usr/share/susemanager:/usr/share/susemanager \
-v /usr/share/salt-formulas/:/usr/share/salt-formulas/ \
-v /srv/susemanager/salt:/srv/susemanager/salt \
-v /srv/salt:/srv/salt \
-v /etc/salt:/etc/salt \
-v /etc/rhn:/etc/rhn \
-v /etc/pki/tls/certs/:/etc/pki/tls/certs/ \
-v /var/log/rhn/:/var/log/rhn/ \
uyuni/server:1.0
```

## Troubleshoot
If the suse manager ports are not open ports in iptables (happens when you run docker and podman in the same machine)
  - find the ip with "podman inspect <ID> | grep IP"
  - the add a rule for each port: `iptables -I CNI-FORWARD -p tcp ! -i cni-podman0 -o cni-podman0 -d <IP> --dport <PORT> -j ACCEPT`


# Stop container

```
docker ps
podman ps
```

```
docker kill -s SIGRTMIN+3 <ID>
```

```
podman kill -s SIGRTMIN+3 <ID>
```

# Set up uyuni

Connect to container terminal

## call setup script
```
/usr/lib/susemanager/bin/mgr-setup -l /var/log/susemanager_setup.log -s
```

re-run the setup command: `rm /root/.MANAGER_SETUP_COMPLETE`

# start existing containers
start it
```
podman start -i <id>
```
## connect to machine terminal

`podman exec -ti <ID> bash`

# Destroy and recreate container

One can destroy the container and recreated it using the same volumes.
This would bring uyuni server to the same state.

Temporary manual steps after destroy and recreate the container:
```
usermod -a -G www tomcat && \
systemctl enable postgresql && \
systemctl start postgresql && \
spacewalk-service enable && \
spacewalk-service start
```
For some reason not yet investigated the user tomcat loses one of the groups and systemd units are disabled.

# Interesting links

https://documentation.suse.com/container/all/html/SLES-container/cha-bci.html

https://osric.com/chris/accidental-developer/2018/12/docker-versus-podman-and-iptables/



---

# TODO

- Some of the folders may not have user data, and should not be saved in volumes
  - analyze all folders from the volumes

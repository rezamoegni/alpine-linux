FROM rezamoegni/alpine-linux:edge

LABEL maintainer="reza.moegni@gmail.com" \
    org.label-schema.name="alpine" \
    org.label-schema.vendor="rezamoegni" \
    org.label-schema.description="Docker image uses openRC as a process supervision on Alpine Linux" \
    org.label-schema.version="edge-openrc" \
    org.label-schema.license="MIT"

RUN echo '@edge http://dl-cdn.alpinelinux.org/alpine/edge/main' >> /etc/apk/repositories \
    && echo '@edgecommunity http://dl-cdn.alpinelinux.org/alpine/edge/community' >> /etc/apk/repositories \
    && echo '@testing http://dl-cdn.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories \
    && apk add --no-cache openrc su-exec ca-certificates wait4ports \
    # Disable getty's
    && sed -i 's/^\(tty\d\:\:\)/#\1/g' /etc/inittab \
    && sed -i \
        # Change subsystem type to "docker"
        -e 's/#rc_sys=".*"/rc_sys="docker"/g' \
        # Allow all variables through
        -e 's/#rc_env_allow=".*"/rc_env_allow="\*"/g' \
        # Start crashed services
        -e 's/#rc_crashed_stop=.*/rc_crashed_stop=NO/g' \
        -e 's/#rc_crashed_start=.*/rc_crashed_start=YES/g' \
        # Define extra dependencies for services
        -e 's/#rc_provide=".*"/rc_provide="loopback net"/g' \
        /etc/rc.conf \
    # Remove unnecessary services
    && rm -f /etc/init.d/hwdrivers \
            /etc/init.d/hwclock \
            /etc/init.d/hwdrivers \
            /etc/init.d/modules \
            /etc/init.d/modules-load \
            /etc/init.d/modloop \
    # Can't do cgroups
    && sed -i 's/\tcgroup_add_service/\t#cgroup_add_service/g' /lib/rc/sh/openrc-run.sh \
    && sed -i 's/VSERVER/DOCKER/Ig' /lib/rc/sh/init.sh

CMD ["/sbin/init"]

COPY assets/root/ /

RUN apk update
RUN apk upgrade
RUN apk --no-cache --update add tor privoxy socat nano bash \
    && mv /etc/tor/torrc.sample  /etc/tor/torrc \
    && echo "ExitNodes {sg}" >> /etc/tor/torrc \
    && echo "StrictNodes 1" >> /etc/tor/torrc \
    && echo "ExcludeNodes {us},{ca},{gb},{uk},{fr},{au},{nz},{cn},{ru},{it},{id},{il}" >> /etc/tor/torrc \
    && echo "forward-socks5 / 0.0.0.0:9050 ." >> /etc/privoxy/config \
    && sed -i 's/listen-address\s*127.0.0.1:8118/listen-address 0.0.0.0:8118/g' /etc/privoxy/config \
    && sed -i \
        -e 's/#SOCKSPort 192.168.0.1:9100/SOCKSPort 0.0.0.0:9050/g' \
        -e 's/#ControlPort 9051/ControlPort 9052/g' \
        /etc/tor/torrc \
    && rc-update add tor \
    && rc-update add privoxy \
    && rc-update add socat 

EXPOSE 9050/tcp 9051/tcp 8118/tcp

FROM alpine:3.5

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY requirements.txt /usr/src/app

# We need openssh for the sftp process, used by sshfs, but for actual sshd we
# use dropbear since it's a lot more forgiving about permissions and the like.
# For some reason pip doesn't install incremental (a Twisted dependency) so do
# so manually. When done, remove unneeded packages for a smaller image.
RUN apk add --no-cache python3 python3-dev openssh gcc libc-dev && \
    ssh-keygen -A && \
    echo -e "ClientAliveInterval 1\nGatewayPorts yes\nPermitEmptyPasswords yes\nPort 8022\nClientAliveCountMax 10\nPermitRootLogin yes\n" >> /etc/ssh/sshd_config && \
    pip3 install --no-cache-dir incremental && \
    pip3 install --no-cache-dir -r requirements.txt && \
    apk del -r gcc libc-dev python3-dev

# Set the permissions necessary to run as a non-root user
RUN chmod -R g+r /etc/ssh && \
    chmod g+w /etc/passwd && \
    chmod -R g+w /usr/src/app

COPY forwarder.py /usr/src/app
COPY socks.py /usr/src/app
COPY . /usr/src/app

# Running as root will now fail due with a permissions error, so default to some
# other UID
USER 1000

CMD /usr/src/app/run.sh

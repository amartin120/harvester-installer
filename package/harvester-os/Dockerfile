ARG BASE_OS_IMAGE
FROM ${BASE_OS_IMAGE} AS micro

# Temporary build stage image
FROM registry.suse.com/bci/bci-base:15.5 AS builder

# Install system packages using builder image that has zypper
COPY --from=micro / /chroot/

COPY SUSEConnect /usr/sbin/SUSEConnect

RUN /usr/bin/zypper --no-refresh --non-interactive remove -t product SLES && \
    rm -rf /etc/zypp/repos.d/* /etc/zypp/services.d/* && \
    SUSEConnect --product "SLE-Micro/5.4/x86_64" --regcode "INTERNAL-USE-ONLY-92f9-9e38" --email "carbide@ranchergovernment.com" && \
    zypper --gpg-auto-import-keys --non-interactive ref && \
    zypper --installroot /chroot -n in --no-recommends dracut-fips openssh-fips && \
    rm -rf /etc/zypp/repos.d/* /etc/zypp/services.d/* /usr/sbin/SUSEConnect

# Main stage using bci-micro as the base image`
FROM micro

COPY --from=builder /chroot/ /

ARG ARCH=amd64
RUN curl -sfL https://github.com/rancher/wharfie/releases/download/v0.6.5/wharfie-${ARCH}  -o /usr/bin/wharfie && chmod +x /usr/bin/wharfie

COPY files/ /
RUN chmod 0600 /system/oem/*

COPY harvester-release.yaml /etc/

RUN mkinitrd

ARG HARVESTER_PRETTY_NAME
RUN sed -i "s/^PRETTY_NAME.*/PRETTY_NAME=\"$HARVESTER_PRETTY_NAME\"/g" /etc/os-release && \
    echo GRUB_ENTRY_NAME="\"$HARVESTER_PRETTY_NAME\"" >> /etc/os-release

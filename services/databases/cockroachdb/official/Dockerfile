FROM registry.access.redhat.com/ubi8/ubi-minimal
ARG BUILDPLATFORM
ARG TARGETPLATFORM
RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM"

# For deployment, we need the following additionally installed:
# tzdata - for time zone functions; reinstalled to replace the missing
#          files in /usr/share/zoneinfo/
# hostname - used in cockroach k8s manifests
# tar - used by kubectl cp
RUN microdnf update -y \
    && rpm --erase --nodeps tzdata \
    && microdnf install tzdata hostname tar -y \
    && rm -rf /var/cache/yum

RUN mkdir /usr/local/lib/cockroach /cockroach /licenses


COPY cockroach.sh cockroach-amd64 cockroach-arm64 /cockroach/
COPY licenses/* /licenses/
# Install GEOS libraries.
COPY libgeos.so libgeos_c.so /usr/local/lib/cockroach/

# Set working directory so that relative paths
# are resolved appropriately when passed as args.
WORKDIR /cockroach/

RUN if [ "$TARGETPLATFORM" = "linux/amd64" ] ; \
    then mv cockroach-amd64 cockroach; \
    else mv cockroach-arm64 cockroach ; \
    fi

# Include the directory in the path to make it easier to invoke
# commands via Docker
ENV PATH=/cockroach:$PATH

ENV COCKROACH_CHANNEL=official-docker

EXPOSE 26257 8080
ENTRYPOINT ["/cockroach/cockroach.sh"]

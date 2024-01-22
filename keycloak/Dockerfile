FROM registry.access.redhat.com/ubi9 AS ubi-micro-build

# COPY apache-maven-3.9.6-bin.zip keycloak-webauthn-conditional-mediation-main.zip /opt/

RUN dnf install -y unzip java-17-openjdk java-17-openjdk-devel && \
    cd /opt && \
    unzip apache-maven-3.9.6-bin.zip && \
    export PATH=/opt/apache-maven-3.9.6/bin:$PATH && \
    unzip keycloak-webauthn-conditional-mediation-main.zip && \
    cd keycloak-webauthn-conditional-mediation-main && \
    mvn clean package

FROM quay.io/keycloak/keycloak:latest as builder
# Enable health and metrics support
ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true

# Configure a database vendor
ENV KC_DB=mysql
COPY --from=ubi-micro-build /opt/keycloak-webauthn-conditional-mediation-main/target/keycloak-webauthn-conditional-mediation.jar /opt/keycloak/providers/
WORKDIR /opt/keycloak
# for demonstration purposes only, please make sure to use proper certificates in production instead
# RUN keytool -genkeypair -storepass password -storetype PKCS12 -keyalg RSA -keysize 2048 -dname "CN=s# erver" -alias server -ext "SAN:c=DNS:localhost,IP:127.0.0.1" -keystore conf/server.keystore
RUN /opt/keycloak/bin/kc.sh build

FROM quay.io/keycloak/keycloak:latest
COPY --from=builder /opt/keycloak/ /opt/keycloak/
COPY --from=ubi-micro-build /mnt/rootfs /

USER root
COPY realm-export.json /etc
RUN pip install uuid
COPY docker-entrypoint.py /usr/bin
RUN chmod +x /usr/bin/docker-entrypoint.py && \
    mkdir -p /opt/keycloak/data/import && \
    chown 1000 /opt/keycloak/data/import
USER 1000
ENTRYPOINT ["/usr/bin/docker-entrypoint.py"]
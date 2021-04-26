FROM jenkins/inbound-agent:latest

ENV TERRAFORM_VERSION=0.12.30
ENV TERRAFORM_SHA256SUM=a646b61232ac0c400ec8cc2c062f4e36b5a9e8515f11f7f5f61eb03fe058f18d

USER root

RUN apt install -y git curl && \
    curl https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip > terraform_${TERRAFORM_VERSION}_linux_amd64.zip && \
    echo "${TERRAFORM_SHA256SUM}  terraform_${TERRAFORM_VERSION}_linux_amd64.zip" > terraform_${TERRAFORM_VERSION}_SHA256SUMS && \
    sha256sum -c terraform_${TERRAFORM_VERSION}_SHA256SUMS && \
    unzip terraform_${TERRAFORM_VERSION}_linux_amd64.zip -d /bin && \
    rm -f terraform_${TERRAFORM_VERSION}_linux_amd64.zip

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && ./aws/install && rm -rf awscliv2.zip && rm -rf ./aws

# RUN curl "https://cacerts.digicert.com/DigiCertTLSRSASHA2562020CA1.crt.pem" -o "DigiCertTLSRSASHA2562020CA1.crt.pem"  &&  keytool -import -trustcacerts -alias Digicert -keystore /usr/local/openjdk-8/jre/lib/security/cacerts -file DigiCertTLSRSASHA2562020CA1.crt.pem -storepass changeit
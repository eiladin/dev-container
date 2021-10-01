# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.194.0/containers/ubuntu/.devcontainer/base.Dockerfile

# [Choice] Ubuntu version: bionic, focal
ARG VARIANT="focal"
FROM mcr.microsoft.com/vscode/devcontainers/base:0-${VARIANT} AS base
ARG PRECOMMIT_VERSION
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends software-properties-common  curl build-essential python3-pip git-crypt \
    && pip install pre-commit==${PRECOMMIT_VERSION}

FROM ubuntu:${VARIANT} as tools
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends software-properties-common curl unzip

FROM tools AS terraform
ARG TERRAFORM_VERSION
ARG GITHUB_TOKEN
RUN curl -H "Authorization: token ${GITHUB_TOKEN}" -H "Accept: application/octet-stream" \
        -sSLJf "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip" -o "terraform.zip" \
    && unzip -qq terraform.zip \
    && chmod +x "./terraform"

FROM tools AS terraform-docs
ARG TERRAFORM_DOCS_VERSION
ARG GITHUB_TOKEN
RUN curl -H "Authorization: token ${GITHUB_TOKEN}" -H "Accept: application/octet-stream" \
        -sSLJfO "https://github.com/terraform-docs/terraform-docs/releases/download/${TERRAFORM_DOCS_VERSION}/terraform-docs-${TERRAFORM_DOCS_VERSION}-linux-amd64.tar.gz" \
    && tar -zxf terraform-docs-${TERRAFORM_DOCS_VERSION}-linux-amd64.tar.gz \
    && chmod +x "./terraform-docs"

FROM tools AS tfsec
ARG TFSEC_VERSION
ARG GITHUB_TOKEN
RUN curl -H "Authorization: token ${GITHUB_TOKEN}" -H "Accept: application/octet-stream" \
      -sSLJf "https://github.com/tfsec/tfsec/releases/download/${TFSEC_VERSION}/tfsec-linux-amd64" \
      -o ./tfsec > /dev/null \
    && chmod +x "./tfsec"

FROM tools AS tflint
ARG TFLINT_VERSION
ARG GITHUB_TOKEN
RUN curl -H "Authorization: token ${GITHUB_TOKEN}" -H "Accept: application/octet-stream" \
      -sSLJfO "https://github.com/terraform-linters/tflint/releases/download/${TFLINT_VERSION}/tflint_linux_amd64.zip" \
    && unzip -qq tflint_linux_amd64.zip \
    && chmod +x "./tflint"

FROM tools AS direnv
ARG DIRENV_VERSION
ARG GITHUB_TOKEN
RUN curl -H "Authorization: token ${GITHUB_TOKEN}" -H "Accept: application/octet-stream" \
      -sSLJf "https://github.com/direnv/direnv/releases/download/${DIRENV_VERSION}/direnv.linux-amd64" -o ./direnv \
    && chmod +x "./direnv"

FROM tools AS kubectl
RUN curl -sSLJO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x ./kubectl

FROM tools AS awscli
RUN curl -sSLJ "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"\
    && unzip awscliv2.zip \
    && ./aws/install

FROM tools AS fzf
ARG FZF_VERSION
RUN curl -sSLJ "https://github.com/junegunn/fzf/releases/download/${FZF_VERSION}/fzf-${FZF_VERSION}-linux_amd64.tar.gz" -o "fzf.tar.gz"\
    && tar zxf fzf.tar.gz \
    && chmod +x ./fzf

FROM tools as krew
RUN apt-get -y install git \
    && groupadd vscode \
    && useradd -s /bin/bash --gid vscode -m vscode
USER vscode
WORKDIR /home/vscode
RUN curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.tar.gz" \
    && tar zxf krew.tar.gz \
    && ./krew-linux_amd64 install krew

FROM base AS final
COPY --from=awscli "/usr/local/aws-cli/" "/usr/local/aws-cli/"
RUN ln -s /usr/local/aws-cli/v2/current/bin/aws /usr/local/bin/aws
RUN ln -s /usr/local/aws-cli/v2/current/bin/aws_completer /usr/local/bin/aws_completer
COPY --from=kubectl "kubectl" "/usr/local/bin/kubectl"
COPY --from=terraform "terraform" "/usr/local/bin/terraform"
COPY --from=terraform-docs "terraform-docs" "/usr/local/bin/terraform-docs"
COPY --from=tfsec "tfsec" "/usr/local/bin/tfsec"
COPY --from=tflint "tflint" "/usr/local/bin/tflint"
COPY --from=direnv "direnv" "/usr/local/bin/direnv"
COPY --from=fzf "fzf" "/usr/local/bin/fzf"
USER vscode
COPY --from=krew "/home/vscode/.krew/" "/home/vscode/.krew"
RUN echo "alias tf='noglob terraform'" | tee -a ~/.zshrc \
    && echo "alias terraform='noglob terraform'" | tee -a ~/.zshrc \
    && echo "alias tf='terraform'" | tee -a ~/.bashrc \
    && echo "eval $(direnv hook zsh)" | tee -a ~/.zshrc \
    && echo "eval $(direnv hook bash)" | tee -a ~/.bashrc \
    && sed -i 's/plugins=(git)/plugins=(git kubectl terraform aws)/g' ~/.zshrc
ENV PATH="/home/vscode/.krew/bin:${PATH}"

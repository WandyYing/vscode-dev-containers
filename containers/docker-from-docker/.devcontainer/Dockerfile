
# Note: You can use any Debian/Ubuntu based image you want. 
FROM mcr.microsoft.com/vscode/devcontainers/base:debian-10

# This Dockerfile adds a non-root user with sudo access. Update the “remoteUser” property in
# devcontainer.json to use it. More info: https://aka.ms/vscode-remote/containers/non-root-user.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Docker script args, location, and expected SHA - SHA generated on release
ARG DOCKER_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/docker-debian.sh"
ARG DOCKER_SCRIPT_SHA="dev-mode"
ARG ENABLE_NONROOT_DOCKER="true"
ARG SOURCE_SOCKET=/var/run/docker-host.sock
ARG TARGET_SOCKET=/var/run/docker.sock

RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    # Verify common dependencies and utilities are installed
    && apt-get -y install --no-install-recommends apt-utils dialog git openssh-client curl less iproute2 procps 2>&1 \
    #
    # Create a non-root user to use if not already available - see https://aka.ms/vscode-remote/containers/non-root-user.
    && if [ $(getent passwd $USERNAME) ]; then \
        # If exists, see if we need to tweak the GID/UID
        if [ "$USER_GID" != "1000" ] || [ "$USER_UID" != "1000" ]; then \
            groupmod --gid $USER_GID $USERNAME \
            && usermod --uid $USER_UID --gid $USER_GID $USERNAME \
            && chown -R $USER_UID:$USER_GID /home/$USERNAME; \
        fi; \
    else \
        # Otherwise ccreate the non-root user
        groupadd --gid $USER_GID $USERNAME \
        && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
        # Add sudo support for the non-root user
        && apt-get install -y sudo \
        && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
        && chmod 0440 /etc/sudoers.d/$USERNAME; \
    fi \
    #
    # Use Docker script from script library to set things up
    && curl -sSL $DOCKER_SCRIPT_SOURCE -o /tmp/docker-setup.sh \
    && ([ "${DOCKER_SCRIPT_SHA}" = "dev-mode" ] || (echo "${DOCKER_SCRIPT_SHA} */tmp/docker-setup.sh" | sha256sum -c -)) \
    && /bin/bash /tmp/docker-setup.sh "${ENABLE_NONROOT_DOCKER}" "${SOURCE_SOCKET}" "${TARGET_SOCKET}" "${USERNAME}" \
    && rm /tmp/docker-setup.sh \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update \
#     && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# Setting the ENTRYPOINT to docker-init.sh will configure non-root access to 
# the Docker socket if "overrideCommand": false is set in devcontainer.json. 
# The script will also execute CMD if you need to alter startup behaviors.
ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
CMD [ "sleep", "infinity" ]
FROM ubuntu:26.04

COPY --from=ghcr.io/jdx/mise:2026.10.4 /usr/local/bin/mise /usr/local/bin/mise

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      build-essential ca-certificates curl docker.io docker-compose-v2 gh git libatomic1 openssh-client sudo vim \
      libbz2-dev libffi-dev liblzma-dev libreadline-dev libsqlite3-dev libssl-dev \
      tk-dev uuid-dev zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

COPY .devcontainer/mise.toml /etc/mise/config.toml

ENV PATH="/usr/local/share/mise/shims:${PATH}"
ENV CARGO_HOME=/usr/local/share/rust/cargo \
    RUSTUP_HOME=/usr/local/share/rust/rustup \
    RUSTUP_INIT_SKIP_PATH_CHECK=yes

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN install -d "${CARGO_HOME}" "${RUSTUP_HOME}" \
    && curl --proto '=https' --tlsv1.2 -fsSL https://sh.rustup.rs | sh -s -- -y --default-toolchain stable --profile minimal --no-modify-path \
    && ln -s "${CARGO_HOME}/bin/rustc" /usr/local/bin/rustc \
    && ln -s "${CARGO_HOME}/bin/cargo" /usr/local/bin/cargo \
    && ln -s "${CARGO_HOME}/bin/rustup" /usr/local/bin/rustup \
    && mise install --system \
    && mise reshim --system

RUN groupmod --new-name thezecter ubuntu \
    && usermod --login thezecter --home /home/thezecter --move-home --gid thezecter ubuntu \
    && install -d -o thezecter -g thezecter /home/thezecter/.codex /home/thezecter/.claude /home/thezecter/.copilot /home/thezecter/.cargo \
    && printf 'thezecter ALL=(ALL) NOPASSWD:ALL\n' > /etc/sudoers.d/thezecter \
    && chmod 0440 /etc/sudoers.d/thezecter

ENV CARGO_HOME=/home/thezecter/.cargo
ENV PATH="/home/thezecter/.cargo/bin:${PATH}"
ENV CLAUDE_CONFIG_DIR=/home/thezecter/.claude

WORKDIR /workspace

USER thezecter

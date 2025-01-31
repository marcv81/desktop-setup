# Install

Install `kubectl`.

    curl -LO https://dl.k8s.io/release/v1.32.0/bin/linux/amd64/kubectl
    chmod 755 kubectl
    sudo chown root: kubectl
    sudo mv kubectl /usr/local/bin/

Install Kind.

    curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-linux-amd64
    chmod 755 kind
    sudo chown root: kind
    sudo mv kind /usr/local/bin/

Install Helm.

    curl -LO https://get.helm.sh/helm-v3.17.0-linux-amd64.tar.gz
    tar -xvzf helm-v3.17.0-linux-amd64.tar.gz
    sudo chown 755 linux-amd64/helm
    sudo mv linux-amd64/helm /usr/local/bin/
    rm -rf linux-amd64/

## Cloud Provider Kind

Build and install.

    go install sigs.k8s.io/cloud-provider-kind@v0.5.0
    sudo install ~/go/bin/cloud-provider-kind /usr/local/bin

Create a user and group.

    sudo groupadd --system cloud-provider-kind
    sudo useradd -g cloud-provider-kind -G docker --system cloud-provider-kind

Define the service in `/etc/systemd/system/cloud-provider-kind.service`.

    [Unit]
    Description=cloud-provider-kind
    After=network.target

    [Service]
    User=cloud-provider-kind
    Group=cloud-provider-kind
    ExecStart=/usr/local/bin/cloud-provider-kind

    [Install]
    WantedBy=multi-user.target

Start the service.

    sudo systemctl daemon-reload
    sudo systemctl start cloud-provider-kind
    sudo systemctl enable cloud-provider-kind

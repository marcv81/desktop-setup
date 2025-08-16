# Alloy

Install Alloy.

    wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg >/dev/null
    echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
    sudo apt update
    sudo apt install alloy

The version was at the time was `1.10.0`.

Give Alloy permissions to read from Docker.

    sudo usermod -a -G docker alloy

Use the `/etc/alloy` configuration from the files directory.

Start and enable the service.

    sudo systemctl start alloy
    sudo systemctl enable alloy

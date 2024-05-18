# Authentication-Encryption-in-Prometheus
# Authentication/Encryption Setup for Prometheus with Node Exporter

## This guide walks you through setting up Authentication and Encryption in Prometheus along with Node Exporter using self-signed certificates and basic authentication.
Certificate Generation for Node Exporter
       
* To start, we need to generate self-signed certificates for Node Exporter.

    ```
    sudo openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout node_exporter.key -out node_exporter.crt -subj 
    "/C=US/ST=California/L=Oakland/O=MyOrg/CN=localhost" -addext "subjectAltName = DNS:localhost"
    ```

* Create a directory for Node Exporter configuration:

    ```
    sudo mkdir /etc/node_exporter
     ```
* Move the generated keys to the Node Exporter directory:

    ```
    sudo mv node_exporter.* /etc/node_exporter/
    ```
    
* Create a config.yml file with TLS configuration:

```yml
  tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key
``` 
* Update permissions for the Node Exporter directory:
  ```bash 
  sudo chown -R node_exporter:node_exporter /etc/node_exporter
  ```
* Update the systemd service of Node Exporter to include TLS configuration:

    ```
    sudo vi /etc/systemd/system/node_exporter.service
    [Unit]
    Description=Node Exporter
    Wants=network-online.target
    After=network-online.target
    
    [Service]
    User=node_exporter
    Group=node_exporter
    Type=simple
    ExecStart=/usr/local/bin/node_exporter --web.config.file="/etc/node_exporter/config.yml"
    
    [Install]
    WantedBy=multi-user.target
    ```
* Reload the systemd daemon and restart Node Exporter:
  ```
     sudo systemctl daemon-reload
     sudo systemctl restart node_exporter
  ```
* Update Prometheus Configuration

* Copy the node_exporter.crt file to the Prometheus server at /etc/prometheus and update permissions:

    ```
    sudo scp /etc/node_exporter/node_exporter.crt aya@192.168.56.110:/etc/prometheus
    sudo chown prometheus:prometheus /etc/prometheus/node_exporter.crt
    ```
    
* Update the Prometheus configuration file (prometheus.yml) with scheme and TLS changes:
```yml
scrape_configs:
  - job_name: 'Linux Server'
    scheme: https
    basic_auth:
      username: prometheus
      password: aya1
    tls_config:
      ca_file: "/etc/prometheus/node_exporter.crt"
      insecure_skip_verify: true
    static_configs:
      - targets: ['192.168.56.111:9100']
 ```

* Restart Prometheus:

     ```bash
     sudo systemctl restart prometheus
     ```
     
* Authentication Setup
* To set up authentication, we need to create a hash of the password using htpasswd:

    ```
    sudo apt-get update && sudo apt install apache2-utils -y
    htpasswd -nBC 12 "" | tr -d ':\n'
    ```
    
* Update the config.yml file for Node Exporter with the username and hashed password.
* Restart Node Exporter:

    ```
    sudo systemctl restart node_exporter
    ```
    
* Update the Prometheus YAML file with basic authentication for the specific job.
Restart Prometheus server:

    ```
    sudo systemctl restart prometheus
    ```
    
* Now you have successfully set up Auth entication/Encryption in Prometheus with Node Exporter.

# FRP-using-AWS-with-EC2
-----

### \#\# How It Works

The process involves two components: the FRP server (`frps`) running on EC2 instance and the FRP client (`frpc`) running on local machine.

1.  **Client Connects:**   local `frpc` initiates a connection to the `frps` on the EC2 instance.
2.  **Tunnel Established:** A stable connection, or tunnel, is maintained between    client and the server.
3.  **Traffic Forwarded:** When a user sends a request to a specific port on    EC2 instance's public IP, `frps` forwards this request through the tunnel to    `frpc`.
4.  **Local Service Responds:**    `frpc` then sends the request to the specified local service (e.g., a web server running on `localhost:3000`). The response travels back along the same path.

-----

### \#\# Step 1: Set Up the AWS EC2 Server (`frps`)

First, you'll configure    EC2 instance to run the FRP server.

#### 1\. Launch and Configure EC2

  * Launch a small EC2 instance, such as a **t2.micro** running **Ubuntu**.
  * Take note of its **Public IPv4 address**.
  * **Crucially, configure its Security Group** to allow incoming traffic on the necessary ports. At a minimum, you'll need to open:
      * **Port `22` (SSH):** To let you manage    instance.
      * **Port `7000` (FRP Bind Port):** For the `frpc` client to connect to the `frps` server.
      * **Port `8080` (Example Web Port):** The public port that users will access. You can change this to any port you need (e.g., `80` for HTTP).

#### 2\. Install and Configure FRP Server

  * SSH into    EC2 instance.

  * Download the latest FRP release for Linux AMD64.

    ```bash
    # Replace the version number with the latest one
    wget https://github.com/fatedier/frp/releases/download/v0.58.1/frp_0.58.1_linux_amd64.tar.gz
    ```

  * Extract the archive and navigate into the directory.

    ```bash
    tar -xzvf frp_0.58.1_linux_amd64.tar.gz
    cd frp_0.58.1_linux_amd64
    ```

  * Edit the server configuration file, `frps.ini`.

    ```bash
    nano frps.ini
    ```

  * Use this minimal, secure configuration. The `token` is **highly recommended** to prevent unauthorized access.

    ```ini
    [common]
    bind_port = 7000
    vhost_http_port = 8080
    token = a_very_secret_and_strong_password
    ```

#### 3\. Run the FRP Server

  * Start the FRP server using the configuration file.

    ```bash
    ./frps -c ./frps.ini
    ```

You should see output indicating the server is listening for connections. For long-term use, you should run `frps` as a systemd service so it starts automatically.

-----

### \#\# Step 2: Configure    Local Client (`frpc`)

Now, configure the FRP client on the local machine where    service is running (e.g.,    laptop).

#### 1\. Install FRP Client

  * Download and extract the appropriate FRP version for    operating system (e.g., Windows, macOS, Linux) from the same GitHub releases page.

#### 2\. Configure FRP Client

  * In the extracted folder, edit the client configuration file, `frpc.ini`.

    ```bash
    nano frpc.ini
    ```

  * Configure it to connect to    EC2 server and expose a local service. This example exposes a local web server running on `localhost:3000`.

    ```ini
    [common]
    # The public IP of    EC2 instance
    server_addr = 3.141.59.26
    server_port = 7000
    # This MUST match the token on the server
    token = a_very_secret_and_strong_password

    [local_web_server]
    type = http
    local_port = 3000
    # Use the EC2 IP as the domain, or a real domain pointed at the IP
    custom_domains = 3.141.59.26
    ```

      * **`server_addr`**:    EC2 instance's public IP.
      * **`local_port`**: The port    local service is running on.
      * **`custom_domains`**: The hostname that `frps` will listen for. Using the EC2's IP address is the simplest way to start.

#### 3\. Run the FRP Client

  * Open a terminal on    local machine, navigate to the FRP folder, and run the client.

    ```bash
    # On Windows
    ./frpc.exe -c ./frpc.ini

    # On macOS/Linux
    ./frpc -c ./frpc.ini
    ```

If the connection is successful, you'll see a log message indicating that the proxy has started.

-----

### \#\# Step 3: Access    Local Service

You're all set\! Open a web browser and navigate to the public address of    EC2 instance on the port you specified as `vhost_http_port` in the `frps.ini` file.

**`http://  _EC2_PUBLIC_IP:8080`**

This request will be securely forwarded to    local development server running on `localhost:3000`. 🎉

# VS Code Remote - Tunnel

Suppose you have GPU-powered containers or PM/VMs running in the cloud and need to access them from your laptop for debugging, development, and monitoring without exposing them to the public internet. 

Tailscale provides a simple, secure, and high-performance solution for this use case.

Another option is VS Code Remote - Tunnel, which is built on top of Microsoft Dev Tunnels. Compared with Tailscale, it is less intrusive and provides convenient access directly from both VS Code and a web browser. The service is free to use, although it comes with limits on the number of active tunnels and the amount of traffic allowed each month. It is particularly well suited for occasional access and lightweight development workflows.

## Documentation and Resources

- [VS Code Remote - Tunnels](https://docs.salad.com/container-engine/tutorials/development-tools/vscode-remote-development)

- [Developing with Remote Tunnels](https://code.visualstudio.com/docs/remote/tunnels)

## Integration

### Step 1: Install and run the VS Code Server on the container

Access the container through the web console, kubectl exec, or any other supported method:

``` bash
# Install the VS Code CLI, which is used to create a secure tunnel between the local and the remote.
curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' -o vscode_cli.tar.gz && \
  tar -xf vscode_cli.tar.gz && \
  mv code /usr/local/bin/code && \
  rm vscode_cli.tar.gz

# Authenticate the tunnel with your GitHub account using the provided sign-in URL.
code tunnel user login --provider github

# Start the VS Code tunnel (ID: 001) in the background
nohup code tunnel --accept-server-license-terms --name 001 &> output.log &
```

### Step 2: Connect to the container using both VS Code Desktop and a browser

```
https://vscode.dev/tunnel/001
```

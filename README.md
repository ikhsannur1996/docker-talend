# Setting up a Docker Container with Xubuntu, Talend Studio, and RDP Access

This guide will walk you through the process of creating a Docker container that runs Xubuntu, installs Talend Studio, and enables Remote Desktop Protocol (RDP) access.

## Prerequisites

1. Docker installed on your system.
2. An RDP client to connect to the container (e.g., Microsoft Remote Desktop on Windows, Remmina on Linux).

## Steps

### Step 1: Create the Dockerfile

```Dockerfile
# Use Ubuntu as the base image
FROM ubuntu:latest

# Install Xubuntu desktop environment and x11vnc
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y xubuntu-desktop x11vnc && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install xrdp for RDP access
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y xrdp && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install required packages for Talend Studio
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    openjdk-11-jre-headless \
    libswt-gtk-4-java \
    unzip \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Set the root password for VNC and xrdp access (change 'your_password' to your desired password)
RUN echo "root:your_password" | chpasswd

# Set up X11 and VNC configurations
RUN mkdir -p /root/.vnc
RUN x11vnc -storepasswd your_password /root/.vnc/passwd

# Download Talend Studio Community Edition (adjust URL as needed for the latest version)
ADD https://download.talend.com/tac/snapshot/Talend-Studio-20211109_1948-V7.3.1.zip /tmp/talend.zip

# Unzip Talend Studio and move it to /opt
RUN unzip /tmp/talend.zip -d /opt \
    && chmod +x /opt/Talend-Studio-*/Talend-Studio-linux-x86_64*

# Set the environment variable for SWT library
ENV SWT_GTK3=0

# Start xrdp server and Xubuntu session with Xvfb (virtual X server) and x11vnc
CMD service xrdp start && Xvfb :0 -screen 0 1024x768x24 && startxfce4 & x11vnc -display :0 -rfbauth /root/.vnc/passwd -forever -shared
```

### Step 2: Build the Docker Image

```bash
docker build -t your-image-name .
```

### Step 3: Run the Docker Container

```bash
docker run -it --rm -p 3389:3389 -e DISPLAY=$DISPLAY your-image-name
```

### Step 4: Connect via RDP

Use your RDP client to connect to the Xubuntu desktop running in the Docker container. Use your local machine's IP address and port `3389` for the RDP connection.

When prompted, enter the password you set in the Dockerfile (replace 'your_password'). Once connected, you will have access to the Xubuntu desktop environment with Talend Studio installed.

---

That's it! You now have a Docker container running Xubuntu, Talend Studio, and accessible via RDP. Enjoy using Talend Studio within the Docker environment! Remember to be cautious when exposing RDP ports to the internet and use this setup only in a trusted and secure environment.

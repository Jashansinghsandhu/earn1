# Anti-Detection Guide for EarnApp Container Setup

This guide explains the anti-detection measures implemented in the `internetIncome.sh` script and provides additional recommendations to prevent EarnApp from detecting that their application is running in a container or on a VPS.

## Overview

EarnApp and similar bandwidth-sharing applications detect containers and VPS environments through various methods:
1. **Hardware fingerprinting** - Checking for virtualization signatures
2. **Network fingerprinting** - Detecting datacenter IP ranges
3. **Environment detection** - Looking for container-specific environment variables
4. **System information** - Checking hostnames, MAC addresses, disk signatures
5. **Timing analysis** - Detecting consistent uptimes and patterns

## Implemented Anti-Detection Measures

The script now includes the following anti-detection features for EarnApp containers:

### 1. Realistic Hostname Generation
- Generates Windows-style hostnames like `DESKTOP-A1B2C3D`, `LAPTOP-X9Y8Z7W`
- Avoids server-like names that indicate VPS or datacenter usage

### 2. Random MAC Address Assignment
- Uses MAC address prefixes from real consumer hardware vendors (Intel, Realtek, etc.)
- Avoids virtualization vendor prefixes (VMware, VirtualBox, etc.)
- Each container gets a unique, realistic MAC address

### 3. Timezone Randomization
- Assigns realistic consumer timezones from common regions
- Helps avoid patterns that indicate automated setups

### 4. Environment Variable Cleanup
- Removes/blanks container-indicator environment variables:
  - `KUBERNETES_SERVICE_HOST`
  - `KUBERNETES_PORT`
  - `DOCKER_HOST`
  - `container`
- Sets realistic locale settings (`LANG`, `LC_ALL`)

### 5. Security Options
- Disables AppArmor confinement (reduces container signatures)
- Disables seccomp filtering (allows more authentic system calls)

### 6. Resource Constraints
- Sets realistic memory limits (512MB)
- Sets realistic CPU limits (1 core)
- Makes container look like a resource-constrained consumer device

## Additional Manual Steps (Highly Recommended)

### 1. VPS/Host-Level Configuration

Before running the script, configure your VPS to look less like a server:

```bash
# Change the VPS hostname to something consumer-like
sudo hostnamectl set-hostname "DESKTOP-$(tr -dc 'A-Z0-9' < /dev/urandom | head -c 7)"

# Disable server services that aren't needed
sudo systemctl disable apache2 nginx postfix 2>/dev/null || true
```

### 2. Use Residential Proxies Correctly

Your residential proxy setup is crucial. Ensure:

1. **Proxy Format**: Use SOCKS5 format for best compatibility:
   ```
   socks5://username:password@ip:port
   ```

2. **Enable proxy DNS**: Set `USE_SOCKS5_DNS=true` in `properties.conf` to route DNS queries through your residential proxy

3. **One proxy per container**: Don't share proxies between multiple EarnApp instances

### 3. Proxy Configuration in properties.conf

Update your `properties.conf` with these settings:

```bash
# Enable proxies
USE_PROXIES=true

# IMPORTANT: Enable DNS through proxy to hide datacenter DNS
USE_SOCKS5_DNS=true

# Optional: Use DNS over HTTPS for additional privacy
USE_DNS_OVER_HTTPS=false

# Disable logs to reduce container footprint
ENABLE_LOGS=false

# Use a consumer-like device name
DEVICE_NAME='HomePC'

# Enable EarnApp
EARNAPP=true
```

### 4. Proxy File Format (proxies.txt)

Add your residential proxies, one per line:

```
socks5://user1:pass1@residential-ip1:port1
socks5://user2:pass2@residential-ip2:port2
# ... up to 20 proxies
```

### 5. Network-Level Recommendations

1. **DNS Leak Prevention**: The script now routes DNS through your proxy when `USE_SOCKS5_DNS=true`

2. **WebRTC Leak Prevention**: EarnApp doesn't use browsers, but if you run other apps, disable WebRTC

3. **IPv6 Disabling** (on host):
   ```bash
   # Disable IPv6 to prevent leaks
   echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
   echo "net.ipv6.conf.default.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```

### 6. Timing and Behavior Recommendations

1. **Stagger container starts**: Don't start all 20 containers at once
   - Start 2-3 containers
   - Wait 30-60 minutes
   - Start the next batch

2. **Keep containers running**: Avoid frequent restarts that look suspicious

3. **Vary uptime patterns**: Occasionally restart containers at random intervals

## Running the Script

After making the recommended changes:

```bash
# First, delete any existing containers
sudo bash internetIncome.sh --delete

# Then start with the new anti-detection features
sudo bash internetIncome.sh --start
```

## Monitoring

Check container status:
```bash
sudo docker ps | grep earnapp
```

Check EarnApp logs (if ENABLE_LOGS=true):
```bash
sudo docker logs <container_name>
```

## Troubleshooting

### If accounts still get banned:

1. **Check proxy quality**: Ensure your residential proxies are truly residential
   - Test at: https://whoer.net/ or https://ipleak.net/

2. **Check for IP reputation**: Some residential IPs may be flagged
   - Try different proxies from your pool

3. **Reduce density**: Don't run too many containers through the same proxy

4. **Contact proxy provider**: Ensure the proxy hasn't been used for violations

### Common Detection Methods EarnApp May Use:

1. **IP Reputation Check**: Even residential IPs can be flagged if previously abused
2. **Connection Pattern Analysis**: Unusual traffic patterns or consistent uptime
3. **Hardware Fingerprint**: Though mitigated, some deep checks may still work
4. **Account Behavior**: Too many devices per account raises flags

## What NOT to Do

1. ❌ Don't use datacenter proxies - they will be detected immediately
2. ❌ Don't run too many containers per account (follow EarnApp's device limits)
3. ❌ Don't use the same proxy for multiple accounts
4. ❌ Don't run containers 24/7 without any breaks (looks unnatural)
5. ❌ Don't use default hostnames or obvious container names
6. ❌ Don't leave ENABLE_LOGS=true in production (creates container signatures)

## Disclaimer

This guide is provided for educational purposes. The modifications help your legitimate residential bandwidth appear as it truly is - residential. If you're using genuine residential proxies with your own bandwidth, these measures simply prevent false positives in detection systems.

Always comply with EarnApp's Terms of Service and only share genuine residential bandwidth.

## Version History

- **v1.0**: Initial anti-detection implementation
  - Realistic hostname generation
  - MAC address spoofing with real vendor prefixes
  - Timezone randomization
  - Environment variable cleanup
  - Security option modifications
  - Resource constraint settings

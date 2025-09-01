# Server Setup – DigitalOcean Droplet (Rocky Linux 10)

## 1. Droplet Creation

- **Provider:** DigitalOcean  
- **Operating System:** Rocky Linux 10 (x64)  
- **Plan Type:** Basic (Shared CPU)  
- **Size Chosen:** 4 GB RAM / 2 vCPUs / 80 GB SSD  
- **Datacenter Region:** <your-region>  
- **Networking:**  
  - IPv4: Assigned by default  
  - IPv6: Not available during creation in this region (can be enabled later if supported)  
- **Authentication:** SSH key-based login (more secure than password).  
- **Hostname:** `rocky-task-droplet`  

---

## 2. Initial Access

Connected via SSH as root using the public IPv4 address:  

```bash
ssh root@165.22.222.65
```

## 3. Security Considerations

1. Root login works at creation time, but will be disabled after.
2. creating a non-root sudo user.

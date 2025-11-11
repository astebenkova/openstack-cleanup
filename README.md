# OpenStack Resource Cleanup Tool

A comprehensive Python tool for safely cleaning up OpenStack resources based on pattern matching. Designed for development environments, CI/CD pipelines, and infrastructure maintenance.

## ✨ Features

- **Pattern-based filtering** - Uses regex patterns to match resource names/descriptions
- **Dry-run mode** - Preview what would be deleted before making changes
- **Comprehensive coverage** - Handles compute, storage, network, load balancer, and advanced services
- **Smart dependency handling** - Deletes resources in the correct order to avoid conflicts
- **Progress monitoring** - Tracks deletion progress and verifies completion
- **Multiple authentication methods** - Supports app credentials, username/password, and openrc files
- **Safety prompts** - Confirms destructive operations unless auto-approved

## 🚀 Quick Start

### 1. Install Dependencies

```bash
pip install openstacksdk tabulate
```

### 2. Set up Authentication

**Option A: Using OpenRC file**
```bash
python3 openstack_cleanup.py -r openrc.sh --dryrun
```

**Option B: Environment variables**
```bash
export OS_AUTH_URL=https://your-openstack.com:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_APPLICATION_CREDENTIAL_ID=your-app-cred-id
export OS_APPLICATION_CREDENTIAL_SECRET=your-app-cred-secret
```

### 3. Preview Resources (Dry Run)

```bash
# Default filter matches ".*test-cluster.*"
python3 openstack_cleanup.py --dryrun

# Custom pattern
python3 openstack_cleanup.py --filter ".*dev-env.*" --dryrun
```

### 4. Cleanup Resources

```bash
# Interactive confirmation
python3 openstack_cleanup.py --filter ".*test-cluster.*"

# Auto-approve (for scripts)
python3 openstack_cleanup.py --filter ".*test-cluster.*" --yes
```

## 📋 Usage Examples

### Development Environment Cleanup
```bash
# Clean up all resources with "dev" in the name
python3 openstack_cleanup.py --filter ".*dev.*" --dryrun
python3 openstack_cleanup.py --filter ".*dev.*" --yes
```

### CI/CD Pipeline Cleanup
```bash
# Clean up build artifacts
python3 openstack_cleanup.py --filter ".*build-[0-9]+.*" --yes
```

### Targeted Resource Cleanup
```bash
# Clean up specific project resources
python3 openstack_cleanup.py --filter ".*project-alpha.*" --dryrun
```

### Using Resource List File
```bash
# Use pre-defined resource list (format: type|name|id per line)
python3 openstack_cleanup.py --file resource_list.log --dryrun
```

## 🛠️ Command Line Options

| Option | Description | Example |
|--------|-------------|---------|
| `-r, --rc` | OpenRC credentials file | `-r openrc.sh` |
| `-f, --file` | Resource list file (type\|name\|id format) | `-f cleanup.log` |
| `-d, --dryrun` | Preview mode - don't delete anything | `--dryrun` |
| `--filter` | Regex pattern for resource names | `--filter ".*test.*"` |
| `-y, --yes` | Auto-approve without prompts | `--yes` |

## 🔧 Authentication Methods

### Application Credentials (Recommended)
```bash
export OS_AUTH_URL=https://openstack.example.com:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_APPLICATION_CREDENTIAL_ID=your-app-cred-id
export OS_APPLICATION_CREDENTIAL_SECRET=your-secret
```

### Username/Password
```bash
export OS_AUTH_URL=https://openstack.example.com:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_USERNAME=your-username
export OS_PASSWORD=your-password
export OS_PROJECT_NAME=your-project
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
```

### OpenRC File
Download from OpenStack Dashboard → Identity → Application Credentials → Download openrc file

## 🗂️ Supported Resources

The tool can clean up the following OpenStack resources:

### Compute Resources
- **Instances/Servers** - Virtual machines and their floating IPs
- **Flavors** - Custom instance types
- **Keypairs** - SSH key pairs
- **Images** - Custom VM images

### Storage Resources
- **Volumes** - Block storage volumes
- **Volume Snapshots** - Point-in-time volume backups

### Network Resources
- **Networks** - Virtual networks and subnets
- **Routers** - Virtual routers and gateways
- **Floating IPs** - Public IP addresses
- **Security Groups** - Firewall rule sets
- **Ports** - Network interfaces

### Load Balancer Resources
- **Load Balancers** - Including listeners, pools, and health monitors

### Advanced Services
- **Heat Stacks** - Orchestration templates
- **DNS Zones** - Designate DNS zones

## ⚠️ Safety Features

### Dry-Run Mode
Always test with `--dryrun` first to preview what would be deleted:
```bash
python3 openstack_cleanup.py --filter ".*test.*" --dryrun
```

### Smart Deletion Order
Resources are deleted in dependency order to avoid conflicts:
1. Advanced Services (Heat stacks, DNS zones)
2. Compute resources (instances, then flavors/keypairs/images)
3. Storage resources (volumes, snapshots)
4. Load balancers
5. Network resources (floating IPs, routers, networks, security groups)

### Confirmation Prompts
Interactive confirmation unless `--yes` is used:
```
Warning: You didn't specify a resource list file as the input. 
The script will delete all resources shown above.
Are you sure? (y/n)
```

### Progress Monitoring
Real-time feedback on deletion progress:
```
*** COMPUTE cleanup
    + INSTANCE test-instance-1 is successfully deleted
    🔍 Verifying INSTANCE 12345678... deletion ✅ DELETED
```

## 🔍 Filter Patterns

The `--filter` option accepts Python regular expressions:

| Pattern | Matches |
|---------|---------|
| `.*test.*` | Any resource with "test" in name/description |
| `^dev-.*` | Resources starting with "dev-" |
| `.*-temp$` | Resources ending with "-temp" |
| `.*(test\|dev\|staging).*` | Resources containing "test", "dev", or "staging" |
| `.*build-[0-9]+.*` | Resources with "build-" followed by numbers |

## 📝 Resource List File Format

When using `--file`, provide a file with one resource per line in format:
```
type|name|id
instances|test-vm-1|12345678-1234-1234-1234-123456789012
networks|test-network|87654321-4321-4321-4321-210987654321
volumes|test-volume|abcdef12-3456-7890-abcd-ef1234567890
```

## 🚨 Important Warnings

### Production Environments
- **NEVER run without `--dryrun` first** in production
- **Double-check filter patterns** to ensure they match only intended resources
- **Test in development** environments before using in production
- **Backup critical data** before running cleanup operations

### Resource Dependencies
- Some resources may take time to fully delete
- The tool waits for completion but may timeout for stuck resources
- Manual intervention may be needed for resources in error states

## 🐛 Troubleshooting

### Authentication Issues
```bash
# Verify credentials
openstack token issue

# Check service catalog
openstack catalog list
```

### Permission Errors
Ensure your user/application credential has sufficient permissions:
- Compute admin (for instances, flavors)
- Network admin (for networks, routers)
- Volume admin (for volumes, snapshots)
- Load balancer admin (for LBaaS resources)

### Stuck Resources
If resources appear stuck in deletion:
1. Check OpenStack service logs
2. Verify no external dependencies exist
3. Try manual deletion through Dashboard
4. Contact OpenStack administrator

### Common Patterns
```bash
# List all matching resources without deletion
openstack server list --name ".*test.*"
openstack network list | grep test

# Check resource status
openstack server show <instance-id>
openstack volume show <volume-id>
```

## 📄 License

Licensed under the Apache License 2.0. See the script header for full license text.

## 🤝 Contributing

1. Test changes in development environments
2. Follow Python PEP 8 style guidelines
3. Add appropriate error handling
4. Update documentation for new features

## 💡 Tips

- Use descriptive patterns to avoid accidental deletions
- Run cleanup regularly to prevent resource accumulation
- Consider automation for CI/CD environments
- Monitor OpenStack quotas and usage

---

**⚠️ Remember: Always run with `--dryrun` first to verify what will be deleted!**

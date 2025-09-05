# Choice.hosting Route 53 Extension - Installation Guide

## Package Information

**Version**: 4.0.0  
**Package**: `choice-hosting-route53-v4.0.0.zip`  
**Size**: 6.2MB  
**Author**: Choice.hosting  

## Installation Methods

### Method 1: Plesk Admin Panel (Recommended)

1. **Download Package**:
   - Download `choice-hosting-route53-v4.0.0.zip`
   - Verify checksum: `sha256sum choice-hosting-route53-v4.0.0.zip`

2. **Install via Plesk**:
   - Login to Plesk admin panel
   - Navigate to **Extensions > My Extensions**
   - Click **Upload Extension**
   - Select `choice-hosting-route53-v4.0.0.zip`
   - Click **Upload and Install**

3. **Configure Extension**:
   - Navigate to **Extensions > Amazon Route 53 - Choice.hosting White-Label DNS Edition**
   - Configure AWS credentials for `choice.hosting-aws-r53` user
   - Set Default Delegation Set ID (e.g., `N3HW3H0W0HBZH6`)
   - **Uncheck** "Manage NS and SOA records"
   - **Check** "Auto-apply hosting tags"
   - **Check** "Turn on Amazon Web Service Route 53"
   - Click **OK**

### Method 2: Manual Installation

1. **Backup Existing Extension**:
   ```bash
   cp -r /usr/local/psa/admin/plib/modules/route53 \
         /usr/local/psa/admin/plib/modules/route53.backup
   ```

2. **Extract and Deploy**:
   ```bash
   # Extract package
   unzip choice-hosting-route53-v4.0.0.zip -d /tmp/route53-new/
   
   # Deploy to Plesk
   cp -r /tmp/route53-new/* /usr/local/psa/admin/plib/modules/route53/
   
   # Set permissions
   chown -R psaadm:psaadm /usr/local/psa/admin/plib/modules/route53/
   chmod -R 755 /usr/local/psa/admin/plib/modules/route53/
   ```

3. **Restart Plesk Services**:
   ```bash
   systemctl restart sw-cp-server
   systemctl restart sw-engine
   ```

## Prerequisites

Before installing, ensure you have:

### 1. AWS Infrastructure Setup
- ✅ **Reusable Delegation Set**: Created in AWS Route 53
- ✅ **White-label A Records**: Pointing to AWS nameservers
- ✅ **IAM User**: `choice.hosting-aws-r53` with restricted policy

### 2. Required IAM Policy
Your IAM user must have this exact policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Route53FullAccess",
      "Effect": "Allow",
      "Action": [
        "route53:CreateHostedZone",
        "route53:DeleteHostedZone", 
        "route53:GetHostedZone",
        "route53:ListHostedZones",
        "route53:ListHostedZonesByName",
        "route53:ChangeResourceRecordSets",
        "route53:ListResourceRecordSets",
        "route53:GetChange",
        "route53:ChangeTagsForResource",
        "route53:ListTagsForResource",
        "route53:GetReusableDelegationSet",
        "route53:ListReusableDelegationSets",
        "route53:GetReusableDelegationSetLimit"
      ],
      "Resource": "*"
    }
  ]
}
```

### 3. System Requirements
- **Plesk**: Onyx (18.0.40) or later
- **PHP**: 7.0+ (extension includes PHP 8.3 compatibility)
- **AWS SDK**: Bundled (3.337.3)
- **Permissions**: Root/administrator access for installation

## Configuration Steps

### 1. AWS Credentials
- **Key**: Access Key ID for `choice.hosting-aws-r53` user
- **Secret**: Secret Access Key for `choice.hosting-aws-r53` user

### 2. Delegation Set Configuration
- **Default Delegation Set**: Your delegation set ID (format: `N3HW3H0W0HBZH6`)
- **Purpose**: Ensures all customer domains use your white-label nameservers

### 3. Critical Settings
- **Manage NS and SOA records**: **MUST BE UNCHECKED**
  - ⚠️ Checking this will interfere with white-label nameserver setup
- **Auto-apply hosting tags**: **SHOULD BE CHECKED**
  - Enables automatic zone categorization and cost tracking
- **Turn on Amazon Web Service Route 53**: **MUST BE CHECKED**
  - Activates the extension

## Verification

### 1. Extension Status
```bash
# Check if extension is loaded
ls -la /usr/local/psa/admin/plib/modules/route53/meta.xml

# Verify version
grep -A1 "<version>" /usr/local/psa/admin/plib/modules/route53/meta.xml
```

### 2. Test Zone Creation
1. Create a test domain in Plesk
2. Check AWS Route 53 console for:
   - ✅ Zone created with correct delegation set
   - ✅ Tags applied: Environment=hosting, ManagedBy=plesk, Type=customer
   - ✅ Customer tag with domain name

### 3. Log Verification
Check Plesk logs for tagging operations:
```bash
# Check for successful tag applications
grep "Applied default tags" /usr/local/psa/var/log/sw-cp-server.log

# Check for tagging errors
grep "Failed to apply tags" /usr/local/psa/var/log/sw-cp-server.log
```

## Troubleshooting

### Installation Issues

**Extension won't upload**:
- Ensure zip file is not corrupted (check SHA256)
- Verify sufficient disk space (55MB+ required)
- Check Plesk admin permissions

**Permission denied during manual install**:
```bash
# Fix ownership and permissions
chown -R psaadm:psaadm /usr/local/psa/admin/plib/modules/route53/
chmod -R 755 /usr/local/psa/admin/plib/modules/route53/
```

### Configuration Issues

**Extension settings not saving**:
```bash
# Restart Plesk services
systemctl restart sw-cp-server
systemctl restart sw-engine

# Clear browser cache and retry
```

**AWS connection errors**:
- Verify IAM user credentials
- Check IAM policy permissions
- Ensure Route53Domains permissions are NOT included (intentional)

### Operational Issues

**Tags not applied to new zones**:
- Check if "Auto-apply hosting tags" is enabled
- Verify IAM user has `route53:ChangeTagsForResource` permission
- Review Plesk logs for tagging errors

**Zones created with wrong nameservers**:
- Verify Default Delegation Set ID is configured
- Check delegation set exists and has capacity
- Ensure "Manage NS and SOA records" is UNCHECKED

## Rollback

If you need to revert to the original extension:

```bash
# Stop Plesk services
systemctl stop sw-cp-server sw-engine

# Restore backup
rm -rf /usr/local/psa/admin/plib/modules/route53
mv /usr/local/psa/admin/plib/modules/route53.backup \
   /usr/local/psa/admin/plib/modules/route53

# Restart services
systemctl start sw-cp-server sw-engine
```

## Support

For installation issues:

1. **Check Requirements**: Ensure all prerequisites are met
2. **Review Logs**: Check Plesk logs for specific error messages
3. **Verify Configuration**: Confirm AWS infrastructure is properly set up
4. **Documentation**: Review [TAGGING-MODIFICATIONS.md](TAGGING-MODIFICATIONS.md) for detailed configuration

## Package Integrity

**SHA256 Checksum**: See `choice-hosting-route53-v4.0.0.sha256`

Verify package integrity:
```bash
sha256sum -c choice-hosting-route53-v4.0.0.sha256
```

## Post-Installation

After successful installation:

1. **Test Zone Creation**: Create test domain to verify functionality
2. **Monitor AWS Costs**: Set up billing alerts for Route 53 charges
3. **Documentation**: Share configuration guide with team
4. **Monitoring**: Set up alerts for failed zone creation/tagging operations
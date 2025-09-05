# Plesk Route 53 Extension - Choice.hosting White-Label DNS Edition

## Overview

This is a customized version of the official Plesk Route 53 extension, specifically modified for white-label DNS hosting businesses. Unlike the standard extension, this version automatically applies consistent AWS tags to hosted zones for improved organization, cost tracking, and zone segmentation between hosting business operations and other AWS resources.

## Key Differences from Official Extension

### Standard Plesk Route 53 Extension
- Creates hosted zones without any tagging
- No zone categorization or business segmentation  
- Requires manual tag application through AWS console
- No cost allocation capabilities for hosting vs personal domains
- Uses broad IAM permissions including Route53Domains

### Choice.hosting White-Label DNS Edition
- **Automatic Tagging**: Every zone gets consistent hosting business tags
- **Zone Segmentation**: Clear separation between hosting and infrastructure zones
- **Cost Allocation Ready**: Built-in tags for AWS billing reports
- **White-label Optimized**: Designed for reusable delegation sets and branded nameservers
- **Restricted IAM Policy**: Focused permissions excluding Route53Domains (manual nameserver management)
- **Configuration Control**: Admin can enable/disable auto-tagging
- **Nameserver Independence**: Assumes pre-configured white-label nameservers and delegation sets
- **Domain Registration Agnostic**: Works with any domain registrar, no AWS domain dependency

## Features Added

### 1. Automatic Tag Application
When creating new hosted zones, the extension automatically applies the following tags:

- **Environment**: `hosting`
- **ManagedBy**: `plesk` 
- **Type**: `customer`
- **Customer**: `[domain-name]` (extracted from zone name)

### 2. Configuration Control
A new checkbox setting "Auto-apply hosting tags" allows administrators to enable/disable the automatic tagging functionality.

### 3. Error Handling
The extension logs successful tag applications and warnings for failed tag operations without disrupting zone creation.

## Files Modified

### 1. `/src/plib/library/Client.php`
- **Line 173-192**: Modified `createHostedZone()` method to call `applyDefaultTags()`
- **Line 315-344**: Added `applyDefaultTags()` private method with tag logic
- **Key Changes**:
  - Checks `autoTagging` setting before applying tags
  - Extracts domain name from zone name for Customer tag
  - Uses `changeTagsForResource` API for tag application
  - Includes comprehensive error handling and logging

### 2. `/src/plib/library/Form/Settings.php`
- **Line 69-83**: Added auto-tagging checkbox and description elements
- **Line 136**: Added setting persistence for `autoTagging` option
- **Key Changes**:
  - New form elements for tagging configuration
  - Default value set to `true` (enabled by default)
  - Descriptive help text explaining tag structure

### 3. `/src/plib/resources/locales/en-US.php`
- **Line 15-16**: Added localization strings for new UI elements
- **Key Changes**:
  - `autoTaggingLabel`: "Auto-apply hosting tags"
  - `autoTaggingDescription`: Detailed explanation of applied tags

## Installation

1. **Backup Current Extension**:
   ```bash
   cp -r /usr/local/psa/admin/plib/modules/route53 /usr/local/psa/admin/plib/modules/route53.backup
   ```

2. **Deploy Modified Extension**:
   ```bash
   cp -r ~/dev/ext-route53/src/* /usr/local/psa/admin/plib/modules/route53/
   ```

3. **Restart Plesk Services**:
   ```bash
   systemctl restart sw-cp-server
   ```

## Configuration

1. Navigate to **Extensions > Amazon Route 53**
2. In the settings form, you'll see a new option: **"Auto-apply hosting tags"**
3. Check the box to enable automatic tagging (enabled by default)
4. Save the configuration

## Tag Structure

All zones created through Plesk will receive these tags:

```
Environment = hosting
ManagedBy = plesk  
Type = customer
Customer = example.com  # (domain name without trailing dot)
```

## White-Label DNS Integration

### Reusable Delegation Set Configuration

This extension is optimized for white-label DNS hosting using a reusable delegation set. The current setup uses:

- **Delegation Set ID**: `N3HW3H0W0HBZH6`
- **White-label Nameservers**: 
  - `ns1.choice.hosting` → `ns-1155.awsdns-16.org`
  - `ns2.choice.hosting` → `ns-339.awsdns-42.com`  
  - `ns3.choice.hosting` → `ns-1912.awsdns-47.co.uk`
  - `ns4.choice.hosting` → `ns-1003.awsdns-61.net`

### Prerequisites and Assumptions

This extension assumes you have **already completed** the white-label DNS setup:

1. **Reusable Delegation Set Created**: You must have created a reusable delegation set in AWS Route 53
2. **White-label A Records Configured**: Your branded nameservers must point to AWS nameservers:
   ```
   ns1.choice.hosting  A  205.251.196.131  ; → ns-1155.awsdns-16.org
   ns2.choice.hosting  A  205.251.193.83   ; → ns-339.awsdns-42.com  
   ns3.choice.hosting  A  205.251.199.120  ; → ns-1912.awsdns-47.co.uk
   ns4.choice.hosting  A  205.251.195.235  ; → ns-1003.awsdns-61.net
   ```
3. **Domain Registrar Nameservers**: Customer domains point to your branded nameservers at the registrar level
4. **IAM User Created**: The restricted `choice.hosting-aws-r53` user exists with proper policy

### Extension Configuration Requirements

1. **Default Delegation Set**: Set to your delegation set ID (e.g., `N3HW3H0W0HBZH6`) in Plesk extension settings
2. **Manage NS and SOA Records**: Should be **UNCHECKED** to preserve white-label setup
3. **Auto-apply Hosting Tags**: **CHECKED** for automatic zone categorization

### What This Extension Does NOT Do

- **Domain Registration**: Does not register domains through AWS Route53Domains
- **Nameserver Management**: Does not modify nameserver records at domain registrars  
- **Delegation Set Creation**: Assumes delegation set is pre-configured
- **White-label Setup**: Assumes A records for branded nameservers are already configured

## Benefits

### 1. Cost Allocation
- Easily track Route 53 costs by hosting business vs personal domains
- Generate cost reports filtered by `Environment=hosting` tag
- Separate billing for customer zones vs infrastructure zones

### 2. Zone Management  
- Quickly identify zones managed by Plesk vs manually created zones
- Filter zones by customer for support and management tasks
- Automatic customer identification through domain-based tagging

### 3. White-label Operations
- Consistent delegation set usage ensures all customer domains use branded nameservers
- Prevents accidental NS record management that could break white-label setup
- Optimized IAM permissions for hosting business operations only

### 4. Automation-Ready
- Tags enable automated scripts and AWS Lambda functions
- Support for automated backups, monitoring, and compliance checks
- Customer-specific automation based on domain tags

### 5. IAM Policy Support
- Works seamlessly with tag-based access controls
- Restricted permissions focused on DNS hosting (no domain registration)
- Maintains compatibility with `choice.hosting-aws-r53` user permissions

## Testing

The extension includes a test script (`test-tagging.php`) that validates:
- Auto-tagging setting behavior
- Tag generation logic
- Zone name to customer name conversion
- Configuration state management

## AWS IAM Policy Requirements

### Required IAM Policy (choice.hosting-aws-r53 user)

This extension requires a specific IAM policy that differs from the standard Plesk Route 53 extension policy. Our policy excludes Route53Domains permissions since nameservers are managed manually for white-label DNS hosting.

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

### Policy Differences from Standard Extension

| Permission | Standard Extension | Choice.hosting Edition | Reason |
|------------|-------------------|------------------------|---------|
| `route53domains:*` | ✅ Included | ❌ **Excluded** | Nameservers managed manually for white-label setup |
| `route53:ChangeTagsForResource` | ❌ Not included | ✅ **Required** | Enables automatic tagging functionality |
| `route53:ListTagsForResource` | ❌ Not included | ✅ **Required** | Tag management and validation |
| `route53:GetReusableDelegationSetLimit` | ❌ Not included | ✅ **Required** | Prevents delegation set limit errors |

### Why Route53Domains Permissions Were Removed

The standard Plesk Route 53 extension includes broad `route53domains:*` permissions, assuming users will manage domain registration through AWS. Our white-label DNS hosting edition **intentionally removes** these permissions because:

1. **Manual Nameserver Management**: White-label DNS hosting requires setting custom nameservers (e.g., `ns1.choice.hosting`) at the domain registrar level, not through AWS APIs

2. **Domain Registrar Independence**: Customers may use any domain registrar (GoDaddy, Namecheap, CloudFlare, etc.) - AWS domain registration is not required

3. **Security Principle**: Least privilege access - only DNS hosting permissions are needed, not domain registration management

4. **Cost Optimization**: Avoids accidental domain registration charges through automated systems

5. **Business Model**: Hosting providers typically don't manage customer domain registrations, only DNS hosting

### IAM User Setup

```bash
# Create IAM user for Plesk Route 53 extension
aws iam create-user --user-name choice.hosting-aws-r53 --path /hosting/

# Attach the policy (assuming you've created it as choice-hosting-route53-policy)
aws iam attach-user-policy \
  --user-name choice.hosting-aws-r53 \
  --policy-arn arn:aws:iam::YOUR-ACCOUNT-ID:policy/choice-hosting-route53-policy

# Create access keys
aws iam create-access-key --user-name choice.hosting-aws-r53
```

## Compatibility

- **AWS IAM Policy**: Uses custom restricted policy optimized for white-label DNS hosting
- **Plesk Version**: Compatible with Plesk Onyx and later
- **PHP Version**: Compatible with PHP 7.0+
- **AWS SDK**: Uses existing AWS SDK for PHP included with extension
- **White-label Setup**: Designed for reusable delegation sets (e.g., N3HW3H0W0HBZH6)

## Troubleshooting

### Common Issues

1. **Tags Not Applied**:
   - Check if auto-tagging is enabled in extension settings
   - Verify IAM user has `route53:ChangeTagsForResource` permission
   - Check Plesk logs for tagging error messages

2. **Extension Configuration Not Saving**:
   - Ensure proper file permissions on modified files
   - Restart `sw-cp-server` service after deployment
   - Clear browser cache and reload Plesk admin panel

3. **Permission Denied Errors**:
   - Verify IAM policy excludes `route53domains:*` permissions (this is intentional)
   - Confirm user has all required Route 53 DNS permissions
   - Check that delegation set ID is correctly configured

4. **Zones Created in Wrong Delegation Set**:
   - Verify "Default Delegation Set" is configured in extension settings
   - Ensure delegation set ID format is correct (no `/delegationset/` prefix)
   - Check that reusable delegation set exists and has available capacity

### Logs Location
Tagging operations are logged to standard Plesk logs:
- Success: `pm_Log::info("Applied default tags to hosted zone: {$hostedZoneId}")`
- Errors: `pm_Log::warn("Failed to apply tags to hosted zone...")`

## Rollback

To revert to original extension:
```bash
rm -rf /usr/local/psa/admin/plib/modules/route53
mv /usr/local/psa/admin/plib/modules/route53.backup /usr/local/psa/admin/plib/modules/route53
systemctl restart sw-cp-server
```

## Future Enhancements

Potential improvements for future versions:
1. Configurable tag templates in admin interface
2. Customer-specific tags based on Plesk subscription data  
3. Tag validation and compliance checking
4. Bulk tagging for existing zones
5. Integration with Plesk billing system for automatic customer identification
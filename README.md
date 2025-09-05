# Plesk Route 53 Extension - Choice.hosting White-Label DNS Edition

[![Apache 2](http://img.shields.io/badge/license-Apache%202-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![AWS Route 53](https://img.shields.io/badge/AWS-Route%2053-orange)](https://aws.amazon.com/route53/)
[![White Label DNS](https://img.shields.io/badge/White%20Label-DNS%20Hosting-green)](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/white-label-name-servers.html)

A specialized version of the Plesk Route 53 extension, optimized for white-label DNS hosting businesses. This edition automatically applies consistent AWS tags for improved cost tracking, zone segmentation, and business operations.

## 🎯 Purpose

Transform your Plesk server into a professional white-label DNS hosting platform using AWS Route 53 infrastructure with:
- **Branded nameservers** (e.g., `ns1.your-brand.com`)  
- **Automatic zone tagging** for cost allocation and management
- **Restricted IAM permissions** focused on DNS hosting (no domain registration)
- **Customer zone segmentation** between hosting business and infrastructure

## ⚡ Quick Start

### Prerequisites
- Plesk Onyx or later
- AWS Account with Route 53 access
- Pre-configured reusable delegation set
- White-label nameserver A records pointing to AWS nameservers

### Installation
```bash
# Backup existing extension
cp -r /usr/local/psa/admin/plib/modules/route53 /usr/local/psa/admin/plib/modules/route53.backup

# Deploy this version
cp -r src/* /usr/local/psa/admin/plib/modules/route53/

# Restart Plesk
systemctl restart sw-cp-server
```

### Configuration
1. Navigate to **Extensions > Amazon Route 53**
2. Enter your AWS credentials for the restricted `choice.hosting-aws-r53` IAM user
3. Set your **Default Delegation Set ID** (e.g., `N3HW3H0W0HBZH6`)
4. **Uncheck** "Manage NS and SOA records" (preserves white-label setup)
5. **Check** "Auto-apply hosting tags" (enables automatic tagging)
6. **Check** "Turn on Amazon Web Service Route 53"

## 🏷️ Automatic Tagging

Every hosted zone created through Plesk automatically receives these tags:

| Tag Key | Tag Value | Purpose |
|---------|-----------|---------|
| `Environment` | `hosting` | Separate hosting business from infrastructure |
| `ManagedBy` | `plesk` | Identify Plesk-managed zones |
| `Type` | `customer` | Categorize as customer zone |
| `Customer` | `domain.com` | Customer identification for billing/support |

## 🔐 AWS IAM Policy

This extension requires a **restricted IAM policy** that excludes Route53Domains permissions:

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

**Note**: Route53Domains permissions are intentionally excluded since nameservers are managed manually for white-label setups.

## 🔍 Key Differences from Official Extension

| Feature | Official Extension | Choice.hosting Edition |
|---------|-------------------|------------------------|
| **Tagging** | Manual only | ✅ Automatic hosting tags |
| **IAM Policy** | Includes Route53Domains | ❌ DNS hosting only |
| **Zone Segmentation** | None | ✅ Business vs infrastructure |
| **Cost Tracking** | Manual setup | ✅ Built-in tag structure |
| **White-label Focus** | Generic DNS | ✅ Hosting provider optimized |

## 📈 Benefits

### Cost Allocation
- Track Route 53 costs by hosting business vs personal domains
- Generate AWS billing reports filtered by `Environment=hosting` tag
- Per-customer cost tracking using domain-based tags

### Zone Management
- Quickly identify Plesk-managed zones vs manual zones
- Filter by customer for support operations
- Automated zone categorization for hosting business

### Business Operations
- Consistent delegation set usage for branded nameservers
- No accidental domain registration charges
- Customer-specific automation capabilities

## 🛠️ White-Label Setup Example

For a hosting business branded as "choice.hosting":

### 1. AWS Delegation Set
```
Delegation Set ID: N3HW3H0W0HBZH6
Nameservers:
  ns-1155.awsdns-16.org
  ns-339.awsdns-42.com
  ns-1912.awsdns-47.co.uk
  ns-1003.awsdns-61.net
```

### 2. White-label A Records
```
ns1.choice.hosting  A  205.251.196.131  ; → ns-1155.awsdns-16.org
ns2.choice.hosting  A  205.251.193.83   ; → ns-339.awsdns-42.com
ns3.choice.hosting  A  205.251.199.120  ; → ns-1912.awsdns-47.co.uk
ns4.choice.hosting  A  205.251.195.235  ; → ns-1003.awsdns-61.net
```

### 3. Customer Domain Setup
Customer points their domain at registrar to:
```
ns1.choice.hosting
ns2.choice.hosting
ns3.choice.hosting
ns4.choice.hosting
```

## 🔧 Troubleshooting

### Common Issues

**Tags not applied?**
- Check if "Auto-apply hosting tags" is enabled
- Verify IAM user has `route53:ChangeTagsForResource` permission
- Check Plesk logs for tagging errors

**Permission denied errors?**
- Confirm IAM policy excludes `route53domains:*` (this is intentional)
- Verify all required Route 53 DNS permissions are present

**Zones in wrong delegation set?**
- Ensure Default Delegation Set ID is configured correctly
- Check delegation set has available capacity (100 zone limit)

## 📚 Documentation

- **[Complete Setup Guide](TAGGING-MODIFICATIONS.md)**: Detailed installation and configuration
- **[AWS White-Label DNS Guide](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/white-label-name-servers.html)**: Official AWS documentation
- **[Plesk DNS Documentation](https://docs.plesk.com/en-US/obsidian/administrator-guide/website-management/dns/)**: Plesk DNS management

## 🤝 Contributing

This is a specialized fork for white-label DNS hosting providers. For general Plesk Route 53 issues, please refer to the [official Plesk repository](https://github.com/plesk/ext-route53).

For white-label DNS hosting specific issues:
1. Check existing issues and documentation
2. Provide AWS IAM policy, delegation set configuration, and Plesk version
3. Include relevant log excerpts from Plesk

## 📄 License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) file for details.

## 🏢 About

Developed for hosting providers who need professional white-label DNS hosting with automated zone management, cost tracking, and customer segmentation capabilities using AWS Route 53 infrastructure.
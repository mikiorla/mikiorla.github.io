---
layout: post
#title: ESXi castore.pem Overview
tags: [ESXi,Certificates]
comments: true
show-avatar: false
#image: /img/ITCS-s.png
---

## ESXi castore.pem Overview

**castore.pem** is a critical file on VMware ESXi hosts, serving as the trusted certificate authority (CA) store for SSL/TLS operations. It contains the root and intermediate CA certificates that the ESXi host trusts for secure communications.

## Common Use Cases

- **Adding Custom CA Certificates:** When integrating ESXi with a custom or enterprise CA, administrators often need to add the root and intermediate certificates to `castore.pem` so the host trusts certificates signed by that CA.
- **Ensuring Certificate Chain Trust:** If the certificate chain (root and intermediates) is incomplete or missing from `castore.pem`, browsers and clients may show trust errors when connecting to ESXi services.


## Location

- The file is located at: `/etc/vmware/ssl/castore.pem`.


## How to Update `castore.pem` on ESXi

**Typical Steps:**

1. **Backup Existing Files:**
    - Before making changes, back up the current `castore.pem`:

```
cp /etc/vmware/ssl/castore.pem /etc/vmware/ssl/castore.pem.bak
```

This allows you to restore the original if needed.
2. **Prepare Your CA Certificate(s):**
    - Obtain the PEM-encoded root and any intermediate CA certificates.
    - If you have a chain, concatenate the certificates in order: intermediate(s) first, then root.
3. **Copy Certificate(s) to ESXi:**
    - Copy your certificate file (e.g., `Root.cer`) to `/etc/vmware/ssl/` on the ESXi host.
4. **Append to `castore.pem`:**
    - Append the new certificate(s) to the existing `castore.pem`:

```
cat Root.cer &gt;&gt; /etc/vmware/ssl/castore.pem
```

You can append multiple root/intermediate certs as needed.
5. **Remove Temporary Files:**
    - Clean up by removing the temporary certificate file:

```
rm /etc/vmware/ssl/Root.cer
```

6. **Persist Changes:**
    - Run the following to persist your changes:

```
/sbin/auto-backup.sh
```

This ensures changes survive a reboot.
7. **Restart or Reconnect Host:**
    - Restart the ESXi host or its management services, then reconnect it to vCenter if needed.

## Key Points and Troubleshooting

- **File Format:** Ensure `castore.pem` is in UTF-8 format to avoid parsing errors.
- **Certificate Chain:** The file should contain all necessary certificates for the trust chain. Missing intermediates can cause browser warnings or failed connections.
- **Rollback:** If issues arise, restore the backup:

```
mv /etc/vmware/ssl/castore.pem.bak /etc/vmware/ssl/castore.pem
```

Then restart management services.


## Summary Table

| File | Purpose | Typical Location |
| :-- | :-- | :-- |
| castore.pem | Trusted CA certificate store (chain) | /etc/vmware/ssl/castore.pem |
| rui.crt | Host SSL certificate | /etc/vmware/ssl/rui.crt |
| rui.key | Host private key | /etc/vmware/ssl/rui.key |

## References

- For detailed step-by-step instructions, see Broadcom's official KB on adding custom certificates.
- For troubleshooting and best practices, see community guides and Dell's documentation.

**In summary:** `castore.pem` is the trusted CA store on ESXi. To add or update trusted certificates, append your CA chain in PEM format to this file, back up originals, and persist changes. This ensures secure, trusted SSL/TLS communication for your ESXi host.

<div style="text-align: center">⁂</div>

https://knowledge.broadcom.com/external/article/317244/adding-custom-certificate-on-esxi-hosts.html

https://knowledge.broadcom.com/external/article/341649/configuring-ca-signed-certificates-for-e.html

https://www.dell.com/support/kbdoc/en-us/000023004/vxrail-how-to-manually-import-vcenter-ssl-certificate-on-esxi

https://www.reddit.com/r/vmware/comments/8z4zal/certificate_chain_on_esxi_nodes/

https://communities.vmware.com/t5/VMware-PowerCLI-Discussions/Error-Replacing-ESXi-SSL-Certificates-and-Key-with-Powercli/td-p/2951244

https://waynehartman.com/posts/update-ssl-certs-in-esxi-server.html

https://stigviewer.com/stig/vmware_vsphere_8.0_esxi/2023-10-11/finding/V-258779

https://www.dell.com/support/kbdoc/de-de/000023004/vxrail-anleitung-zur-manuellen-import-von-vcenter-ssl-zertifikat-auf-esxi


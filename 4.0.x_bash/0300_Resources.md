# ASNs
A 2-byte ASN is a 16-bit number. This format provides for 65,536 ASNs (0 to 65535). Internet Assigned Numbers Authority (IANA) reserved 1,023 of them (64512 to 65534) for private use.<br>
A 4-byte ASN is a 32-bit number. Internet Assigned Numbers Authority (IANA) reserved a block of 94,967,295 ASNs (4200000000 to 4294967294) for private use.<br>
Source: https://www.arin.net/resources/guide/asn/<br>
<br>
<b>Schema:</b>
<span style="color:gray">42</span> <span style="color:blue">XXXX</span> <span style="color:green">YYYY</span><br>
<span style="color:gray">42</span> = Static - for private use<br>
<span style="color:blue">XXXX</span> = DataCenter ID - DC1=0001<br> 
<span style="color:green">YYYY</span> = Device ID<br>
<br>
<b>DC1 Example:</b><br>
[ASNs – Suer Spines (ID 4200010010 - 4200010019)](#asns_super_spines)<br>
[ASNs – Spines (ID 4200010020 - 4200010029)](#asns_spines)<br>
[ASNs – Leafs (ID 4200010030 - 4200019999)](#asns_leafs)<br>
[ASNs - Externals (65000 - 65099)](#asns_externals)<br>

# IP-Pools
Middle-Size DC require private IP range /21 for IP-Pools<br>
<br> 
<b>DC1 Example:</b><br>
|- <b>10.255.0.0/21</b> (IP Pool DC1)<br>
|-- <b>10.255.0.0/22</b> (Loopbacks) <br>
&nbsp;&nbsp;&nbsp;|--> [10.255.0.0/28 - (16 IPs) - Super Spines](#ip_superspines)<br>
&nbsp;&nbsp;&nbsp;|--> [10.255.0.16/28 - (16 IPs) - Spines](#ip_spines)<br>
&nbsp;&nbsp;&nbsp;|--> [10.255.0.32/27 - (32 IPs)   - Leafs](#ip_leafs)<br>
&nbsp;&nbsp;&nbsp;|--> [10.255.0.64/26 - (64 IPs)   - Leafs](#ip_leafs)<br>
&nbsp;&nbsp;&nbsp;|--> [10.255.0.128/25 - (128 IPs) - Leafs](#ip_leafs)<br>
&nbsp;&nbsp;&nbsp;|--> 10.255.1.0/24 - (256 IPs) - Reserved<br>
&nbsp;&nbsp;&nbsp;|--> [10.255.2.0/24 - (256 IPs) - Reserved<br>
&nbsp;&nbsp;&nbsp;|--> [10.255.2.0/24 - (256 IPs) - Reserved<br>
|-- <b>10.255.4.0/22</b> (Link IPs)<br>
&nbsp;&nbsp;&nbsp;|--> [10.255.4.0/25 - (64 links) - To Generic Underlay (External)](#ip_link_generic)<br>
&nbsp;&nbsp;&nbsp;|--> [10.255.4.128/26 - (32 links) - Spines<>Superspines](#ip_link_spines2superspines)<br>
&nbsp;&nbsp;&nbsp;|--> 10.255.4.192/26 - (32 links) Reserverd<br>
&nbsp;&nbsp;&nbsp;|--> 10.255.5.0/24 - (128 links) Reserverd<br>
&nbsp;&nbsp;&nbsp;|--> [10.255.6.0/23 - (256 links) - Spines<>Leafs](#ip_link_spines2leafs)<br>
|-- <b>169.254.0.0/16</b> (VNI Loopbacks)  <br>
&nbsp;&nbsp;&nbsp;|--> [169.254.0.0/16 - (65534 IPs) - VNI Loopbacks](#ip_vni) <font color="red">\*</font><br>
<br>
<font color="red">\*</font>&nbsp; Please note that VNI Loopbacks are visibile inside VNI. You needs 1xIP per VNI / per Leaf.<br>
VNI Loopbacks can overlap between VNI - so you can use same range ie. 169.254.254.0/23 for each VNI.<br>
VNI Loopbacks cannot overlap with customer IP Network - thats is why I propose to use not routable link-local IPv4 address based on RFC 3927. <br>

# VNI
A VNI is a 24-bit number that is assigned to a VLAN to distinguish it from other VLANs that are on a VXLAN tunnel interface (VTI). VNI values range from 1 to 16777215 in decimal notation and from 0.0. 1 to 255.255.<br>
<b>IMPORTANT:</b> <i>Apstra allows setting VNI in range: <b>4096 - 16'777'214</b></i><br>

<b>Schema:</b><br>
<span style="color:blue">XXX</span> <span style="color:green">Y YYYY</span><br>
<span style="color:blue">XXX</span> = DataCenter ID (1-167)<br> 
<span style="color:green">Y YYYY</span> = VNI Customer ID (0-99999)<br>

<b>DC1 Example:</b><br>
[VNI CustomerA: 1 0 0001](#vni_dc1) -> <span style="color:blue">1</span><span style="color:green">00001</span> <br>
VNI CustomerB: 1 0 0002 -> <span style="color:blue">1</span><span style="color:green">00002</span> <br>
VNI CustomerC: 1 0 0003 -> <span style="color:blue">1</span><span style="color:green">00003</span> <br>

<br>
<br>
<br>

## ------------------------------------------------------------------------------------------------
<a name="asns_super_spines"></a>
## ASNs – Suer Spines (ID 10-19)
```bash
cat <<EOT > /tmp/resources_asn-pools_DC1-SuperSpines.json
{
      "id": "asn_dc1_superspines",
      "display_name": "DC1-SuperSpines",
      "tags": [
        "DC1"
      ],
      "ranges": [
        {
          "status": "pool_element_available",
          "first": 4200010010,
          "last": 4200010019
        }
      ]
    }
EOT
```

```bash
curl  -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/asn-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_asn-pools_DC1-SuperSpines.json
```

<a name="asns_spines"></a>
## ASNs – Spines (ID 20-29)
### API POST (create) 
```bash
cat <<EOT > /tmp/resources_asn-pools_DC1-Spines.json
{
      "id": "asn_dc1_spines",
      "display_name": "DC1-Spines",
      "tags": [
        "DC1"
      ],
      "ranges": [
        {
          "status": "pool_element_available",
          "first": 4200010020,
          "last": 4200010029
        }
      ]
    }
EOT
```

```bash
curl  -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/asn-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_asn-pools_DC1-Spines.json
```

<a name="asns_leafs"></a>
## ASNs – Leafs (ID 30-9999)
### API POST (create) 
```bash
cat <<EOT > /tmp/resources_asn-pools_DC1-Leafs.json
{
      "id": "asn_dc1_leafs",
      "display_name": "DC1-Leafs",
      "tags": [
        "DC1"
      ],
      "ranges": [
        {
          "status": "pool_element_available",
          "first": 4200010030,
          "last": 4200019999
        }
      ]
    }
EOT
```

```bash
curl -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/asn-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_asn-pools_DC1-Leafs.json
```


<a name="asns_externals"></a>
## ASNs - Externals (65000 - 65099)
### API POST (create) 
```bash
cat <<EOT > /tmp/resources_asn-pools_DC1-Externals.json
{
      "id": "asn_dc1_externals",
      "display_name": "DC1-Externals",
      "tags": [
        "DC1"
      ],
      "ranges": [
        {
          "status": "pool_element_available",
          "first": 65000,
          "last": 65099
        }
      ]
    }
EOT
```

```bash
curl -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/asn-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_asn-pools_DC1-Externals.json
```



## ------------------------------------------------------------------------------------------------
<a name="ip_superspines"></a>
## DC1-Loopbacks-10.255.0.0/24 (Super Spines)
### API POST (create)
```bash
cat <<EOT > /tmp/resources_ip-pools_DC1-Loopbacks-SuperSpines.json
{
      "id": "ip_dc1_loopbacks_superspines",
      "display_name": "DC1-Lo-SuperSpines",
      "status": "not_in_use",
      "subnets": [
        {
          "status": "pool_element_available",
          "network": "10.255.0.0/28"
        }
      ],
      "tags": [
        "DC1"
      ]
}
EOT
```

```bash
curl  -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/ip-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_ip-pools_DC1-Loopbacks-SuperSpines.json
```
<a name="ip_spines"></a>
### API POST (create)  - Spines
```bash
cat <<EOT > /tmp/resources_ip-pools_DC1-Loopbacks-Spines.json
{
      "id": "ip_dc1_loopbacks_spines",
      "display_name": "DC1-Lo-Spines",
      "status": "not_in_use",
      "subnets": [
        {
          "status": "pool_element_available",
          "network": "10.255.0.16/28"
        }
      ],
      "tags": [
        "DC1"
      ]
}
EOT
```

```bash
curl  -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/ip-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_ip-pools_DC1-Loopbacks-Spines.json
```

<a name="ip_leafs"></a>
### API POST (create)  - Leafs
```bash
cat <<EOT > /tmp/resources_ip-pools_DC1-Loopbacks-Leafs.json
{
      "id": "ip_dc1_loopbacks_leafs",
      "display_name": "DC1-Lo-Leafs",
      "status": "not_in_use",
      "subnets": [
        {
          "status": "pool_element_available",
          "network": "10.255.0.32/27"
        },
        {
          "status": "pool_element_available",
          "network": "10.255.0.64/26"
        },
        {
          "status": "pool_element_available",
          "network": "10.255.0.128/25"
        }
      ],
      "tags": [
        "DC1"
      ]
}
EOT
```

```bash
curl  -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/ip-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_ip-pools_DC1-Loopbacks-Leafs.json
```

<a name="ip_vni"></a>
### API POST (create)  - VNI Loopbacks
```bash
cat <<EOT > /tmp/resources_ip-pools_DC1-VNI-Loopbacks.json
{
      "id": "ip_dc1_vni_loopbacks",
      "display_name": "DC1-VNI-Loopbacks",
      "status": "not_in_use",
      "subnets": [
        {
          "status": "pool_element_available",
          "network": "169.254.0.0/16"
        }
      ],
      "tags": [
        "DC1"
      ]
}
EOT
```
```bash
curl  -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/ip-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_ip-pools_DC1-VNI-Loopbacks.json
```


<a name="ip_link_generic"></a>
## DC1-Links - To Generic Underlay (external)
### API POST (create) 
```bash
cat <<EOT > /tmp/resources_ip-pools_DC1-Links-To-Generic-Underlay.json
{
      "id": "ip_dc1_links_to_generic_underlay",
      "display_name": "DC1-Links-To-Generic-Underlay",
      "subnets": [
        {
          "status": "pool_element_available",
          "network": "10.255.4.0/25"
        }
      ],
      "tags": [
        "DC1"
      ]
}
EOT
```

```bash
curl -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/ip-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_ip-pools_DC1-Links-To-Generic-Underlay.json
```

<a name="ip_link_spines2superspines"></a>
## DC1-Links - Spines<>Superspines
### API POST (create) 
```bash
cat <<EOT > /tmp/resources_ip-pools_DC1-Spines2Superspines.json
{
      "id": "ip_dc1_links_spines2superspines",
      "display_name": "DC1-Links-Spines2Superspines",
      "subnets": [
        {
          "status": "pool_element_available",
          "network": "10.255.4.128/26"
        }
      ],
      "tags": [
        "DC1"
      ]
}
EOT
```

```bash
curl -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/ip-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_ip-pools_DC1-Spines2Superspines.json
```

<a name="ip_link_spines2leafs"></a>
## DC1-Links - Spines<>Leafs
### API POST (create) 
```bash
cat <<EOT > /tmp/resources_ip-pools_DC1-Spines2Leafs.json
{
      "id": "ip_dc1_links_spines2leafs",
      "display_name": "DC1-Links-Spines2Leafs",
      "subnets": [
        {
          "status": "pool_element_available",
          "network": "10.255.6.0/23"
        }
      ],
      "tags": [
        "DC1"
      ]
}
EOT
```

```bash
curl -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/ip-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_ip-pools_DC1-Spines2Leafs.json
```



## ------------------------------------------------------------------------------------------------
<a name="vni_dc1"></a>
## VNI DC1 (100000 - 199999)
### API POST (create)
```bash
cat <<EOT > /tmp/resources_vni-pools_DC1.json
{
  "id": "vni_dc1",
  "display_name": "DC1",
  "ranges": [
    {
      "status": "pool_element_available",
      "first": 10000,
      "last":  19999
    }
  ],
  "tags": [
    "DC1"
  ]
}
EOT
```

```bash
curl -H "AuthToken: $token" \
  -k -X POST "https://$apstra_ip/api/resources/vni-pools" \
  -H  "accept: application/json" \
  -H  "content-type: application/json" \
  -d @/tmp/resources_vni-pools_DC1.json
```
# Getting Started with telephony42 (Asterisk)

This guide will help you quickly deploy a working Asterisk PBX. By the end of this tutorial, you will be able to make and receive calls in telephony42.

## Prerequisites

Before configuring Asterisk, ensure you have:

1. An allocated telephony42 (`+042`) number prefix registered in the registry (e.g., `+04240000`).
2. DNS NAPTR records correctly configured on your zone in `tel.dn42`.

Example of wildcard NAPTR record to route any calls to your PBX at `pbx.yourdomain.dn42` with SIP port `5060`:

```bind
*.0.0.0.0.4.2.4.0.tel.dn42. IN NAPTR 100 10 "u" "E2U+sip" "!^(.*)$!sip:\\1@pbx.yourdomain.dn42:5060!" .
```

## Installation notes

### Debian / Ubuntu

Asterisk is available in the official repositories. (For Debian, you need to install packages from `sid` repository)

```sh
apt update
apt install asterisk
```

### Alpine

```sh
apk add asterisk
```

### Docker (`docker compose`)

```yaml
services:
  asterisk:
    image: andrius/asterisk:stable
    container_name: asterisk42
    network_mode: host
    volumes:
      - ./config:/etc/asterisk
    restart: unless-stopped
```

# Example configuration

Asterisk relies on many configuration files. This guide will focus only on two main config files:
- `pjsip.conf`: Network, Endpoints, and Authentication
- `extensions.conf`: Dialplan and Routing

When copying the configurations below, replace the following variables (remove the brackets):
* `<OWN_IPv4>`: Your dn42 IPv4 address (e.g., `172.20.X.X`)
* `<OWN_IPv6>`: Your dn42 IPv6 address (e.g., `fdXX::X`)
* `<OWN_PREFIX>`: Your telephony prefix (e.g., `+04240000`)
* `<EXT_NAME>`: Name for your extension (e.g., `0001` or `my-phone`)
* `<FULL_NUMBER>`: The full `+042` number for your extension (e.g., `+042400000001`)
* `<SECRET_PASS>`: A strong password

## PJSIP (`pjsip.conf`)

This file defines how Asterisk listens to the network, sets up templates for devices, creates your local extension, and prepares an anonymous endpoint to catch incoming ENUM calls.

```ini
[global]
type=global
; verified peers first
endpoint_identifier_order=auth_username,username,ip

; ==========================================
; Transports
; ==========================================

[transport-udp4]
type=transport
protocol=udp
bind=<OWN_IPv4>:5060
local_net=172.20.0.0/14
local_net=172.31.0.0/16
local_net=10.0.0.0/8

[transport-udp6]
type=transport
protocol=udp
bind=[<OWN_IPv6>]:5060
local_net=fd00::/8

; ==========================================
; Templates
; ==========================================

[phone-template](!)
type=endpoint
context=context-local          ; Incoming calls from this phone go here
allow=!all,ulaw,alaw           ; Use standard audio formats
rtp_symmetric=yes
force_rport=yes
rewrite_contact=yes
direct_media=no                ; Proxy media through PBX to prevent NAT issues
trust_id_inbound=yes
send_pai=yes

; ==========================================
; Local Extensions
; ==========================================

; Replace variables to create an extension names <EXT_NAME>
[<EXT_NAME>](phone-template)
auth=<EXT_NAME>
aors=<EXT_NAME>
callerid="My Name" <<FULL_NUMBER>>

[<EXT_NAME>]
type=auth
auth_type=userpass
username=<EXT_NAME>
password=<SECRET_PASS>

[<EXT_NAME>]
type=aor
max_contacts=3   ; Allow 3 devices to register simultaneously
remove_existing=yes

; ==========================================
; Incoming calls via ENUM
; ==========================================

[peer-enum-inbound]
type=endpoint
context=context-enum   ; Send unknown incoming calls to ENUM context
allow=!all,ulaw,alaw           ; Use standard audio formats
rtp_symmetric=yes
force_rport=yes
rewrite_contact=yes
direct_media=no

[peer-enum-inbound-id]
type=identify
endpoint=peer-enum-inbound
; match all dn42 subnets
match=172.20.0.0/14
match=172.31.0.0/16
match=10.0.0.0/8
match=fd00::/8

; ==========================================
; Outgoing calls via ENUM
; ==========================================

[peer-enum-outbound]
type=endpoint
allow=!all,ulaw,alaw           ; Use standard audio formats
send_pai=yes
```

## Dialplan (`extensions.conf`)

The file tells Asterisk how to route numbers. It is divided into serveral contexts.

```ini
[globals]
; (Optional) Global variables can go here

; ==========================================
; Inbound Routing
; ==========================================

; Calls originating from your own network.
[context-local]

; 4 digits -> call local extension
exten => _XXXX,1,Goto(ext-local,${EXTEN},1)

; starts with 042 -> add + and route out
exten => _042X.,1,Goto(ext-routing,+${EXTEN},1)

; standard +042 dialing -> route out
exten => _+042X.,1,Goto(ext-routing,${EXTEN},1)

; Catch-all -> add +042 and route out
exten => _X!,1,Goto(ext-routing,+042${EXTEN},1)


; Calls originating from external networks via ENUM.
[context-enum]

; starts with 00 -> replace with + and route out
exten => _00X!,1,Goto(ext-routing,+${EXTEN:2},1)

; starts with + -> route out
exten => _+X!,1,Goto(ext-routing,${EXTEN},1)

; Catch-all -> add + and route out
exten => _X!,1,Goto(ext-routing,+${EXTEN},1)


; ==========================================
; Outbound & Local Routing
; ==========================================

; Calls destined for local networks.
[ext-local]
; route <OWN_PREFIX>0001 calls to your local extension <EXT_NAME>
exten => 0001,1,Dial(PJSIP/<EXT_NAME>,30)  ; 


; Inter-PBX routing for both inbound and outbound calls.
[ext-routing]

; route calls destined for your prefix
exten => _<OWN_PREFIX>.,1,Goto(ext-local,${EXTEN:9},1)

; other external calls -> do ENUM lookup on tel.dn42
exten => _+042X.,1,Set(TARGET_URI=${ENUMLOOKUP(${EXTEN},sip,,,tel.dn42)})
 ; if found, dial via our outbound endpoint
 same => n,ExecIf($["${TARGET_URI}"!=""]?Dial(PJSIP/peer-enum-outbound/sip:${TARGET_URI},60))
 ; if unallocated or failed, naturally hangup
 same => n,Hangup()
```

Apply your configuration by entering the Asterisk CLI:

```sh
asterisk -rx 'core reload'
```

# Setting up your extension

You can now connect a softphone (like MicroSIP, Zoiper, or Linphone) or a hardware IP Phone/ATA using the following details:

* Domain / Server: pbx.yourdomain.dn42 / `<OWN_IPv4>` or `<OWN_IPv6>`
* Port: 5060 (UDP)
* Username: `<EXT_NAME>`
* Password: `<SECRET_PASS>`

Once connected, ask a friend in telephony42 to call your `<FULL_NUMBER>`.

*Tip: Install and use `sngrep -c` in your PBX's terminal to visually monitor SIP traffic if calls fail.*

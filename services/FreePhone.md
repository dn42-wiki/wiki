# What's FreePhone?
Where's the point in using a phone flat just for a single person? !FreePhone is a project aimed to develop a VPN wide SIP phone service. Calling german landline is possible at the moment, as well as local participants (eg. maxx).

## How does this work?
### Public proxy
Set up your softphone or hardware implementation to use:
 * SIP-Proxy/Proxy domain: maxx.spaceboyz.net (SRV-Record) 
 * Username/Account/Login: vpn
 * Password: vpn    
The proxy is strictly outbound, registration is impossible and unintended.

## Special needs
Just contact me if you like to use your SIP hardware (eg. Fritz!Box FON). You'll get a special account allowing registrations plus a local extension.

## Restrictions
 * Any call under the terms of the flatrate is allowed, so to speak: no mobile phones or pr0n calls
 * One call at a time for FreePhone users (stupid bandwidth restrictions :/).
 * Internal calls are more or less unrestricted.
 * alaw/ulaw are disallowed for bandwidth reasons

## Additional extensions
| **Extension** | **Target** |
|---|---|
| maxx | myself, almost anywhere wireless lan is availiable |
| grim | sometimes, sometimes not |
| equinox | i think nokia prevents but you may try |
| helios | did not connect for some time now |

If you like listening to german news, dial 787326353 (Vanity: STREAMDLF). Just contact me in case you want more.

## Configuration examples
Just look at the german version, you'll get the idea.

## What's next?
### Real dn42 phone system
If i'm bored some day i might implement the following:
 * SIP extensions for every participant
 * Voicemail
 * Funny games
 * FreePhone integration (maybe with redundancy)
 * ...

If someone is willing to experiment we could try allowing reinvites. This way all SIP endpoints inside the VPN could connect their media streams directly, thus saving bandwidth and raising call quality.

## Latest changes
 * G.729 now is the preferred codec because of bandwith issues
 * My "Homezone" works perfectly, moving with me
  * Phone #: +493727/959023
  * Sipgate: 5884293
  * SIP: maxx(at)maxx.spaceboyz.net
 * Transcoding from/into G.729 works fine now, thanks to some precompiled versions for asterisk.

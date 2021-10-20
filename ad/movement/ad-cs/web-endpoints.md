# Web endpoints

## Theory

In [their research papers](https://posts.specterops.io/certified-pre-owned-d95910965cd2), [Will Schroeder](https://twitter.com/harmj0y) and [Lee Christensen](https://twitter.com/tifkin\_) found a domain escalation vector based on web endpoints vulnerable to NTLM relay attacks. The escalation vector was dubbed [ESC8](https://posts.specterops.io/certified-pre-owned-d95910965cd2#48bd).

> AD CS supports several HTTP-based enrollment methods via additional server roles that administrators can optionally install \[(The certificate enrollment web interface, Certificate Enrollment Service (CES), Network Device Enrollment Service (NDES)).]
>
> \[...]
>
> These HTTP-based certificate enrollment interfaces are all vulnerable to NTLM relay attacks. Using NTLM relay, an attacker can impersonate an inbound-NTLM-authenticating victim user. While impersonating the victim user, an attacker could access these web interfaces and request a client authentication certificate based on the "User" or "Machine" certificate templates.
>
> ([specterops.io](https://posts.specterops.io/certified-pre-owned-d95910965cd2#5c3c))

This attack, like all NTLM relay attacks, requires a victim account to authenticate to an attacker-controlled machine. An attacker can coerce authentication by many means, see MITM and coerced authentication coercion techniques. Once the incoming authentication is received by the attacker, it can be relayed to an AD CS web endpoint.

Once the relayed session is obtained, the attacker poses as the relayed account and can request a client authentication certificate (e.g. using the default **User** or **Machine/Computer **templates, usually enabled).

This allows for lateral movement, and in some cases privilege escalation if the relayed user had powerful privileges (e.g., domain controllers or Exchange servers, domain admins etc.).

## Practice

{% tabs %}
{% tab title="UNIX-like" %}
From UNIX-like systems, [Impacket](https://github.com/SecureAuthCorp/impacket)'s [ntlmrelayx](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py) (Python) can be used to conduct the ESC8 escalation scenario.

```python
ntlmrelayx -t "http://CA/certsrv/certfnsh.asp" --asp --template "Template name"
```

The certificate template flag (i.e. `--template`) can either be left blank (default to **Machine** at the time of writing, October 20th 2012) or chosen among the certificate templates that fill the requirements.&#x20;



{% tabs %}
{% tab title="UNIX-like" %}
From UNIX-like systems, [Certipy](https://github.com/ly4k/Certipy) (Python) can be used to enumerate info about the CAs, including the "User Specified SAN" flag state.

```python
certipy 'domain.local'/'user':'password'@'domaincontroller' find | grep "User Specified SAN"
```

{% hint style="info" %}
By default, Certipy uses LDAPS, which is not always supported by the domain controllers. The `-scheme` flag can be used to set whether to use LDAP or LDAPS.
{% endhint %}

The same "find" command can be used to enumerate information regarding the certificate templates (EKUs allowing for authentication, allowing low-priv users to enroll, etc.).

```bash
certipy 'domain.local'/'user':'password'@'domaincontroller' findhttps://github.com/SecureAuthCorp/impacket
```
{% endtab %}
{% endtabs %}
{% endtab %}

{% tab title="Windows" %}
From Windows systems, the [Certify](https://github.com/GhostPack/Certify) (C#) tool can be used to enumerate enabled web endpoints (both HTTP and HTTPS).

```batch
Certify.exe cas
```

{% hint style="warning" %}
If web endpoints are enabled, switch to UNIX because at the time of writing (October 20th, 2021), I don't know how to conduct the ESC8 abuse from Windows.
{% endhint %}
{% endtab %}
{% endtabs %}

## Resources

{% embed url="https://www.exandroid.dev/2021/06/23/ad-cs-relay-attack-practical-guide" %}

{% embed url="https://dirkjanm.io/ntlm-relaying-to-ad-certificate-services" %}

{% embed url="https://posts.specterops.io/certified-pre-owned-d95910965cd2" %}
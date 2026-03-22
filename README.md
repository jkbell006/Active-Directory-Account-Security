# Active-Directory-Account-Security
Demonstrates a kerberoasting attack and mitigation strategies.


Introduction

CyberSecurity is essential for a modern business environment, with various attacks posing risks. This project aims to demonstrate a known attack method on the domain controller through Microsoft Active Directory’s (AD) kerberos ticketing system.

I will also look at various detection strategies using Defense in Depth (DiD.) While security features exist, it's critical to understand the specific risks, such as the potential to obtain password hashes for offline cracking and the heightened risk when connected to SQL servers, which can lead to data compromise or Denial of service. 

	This emphasizes the need for a continuous attitude of risk analysis and mitigation strategies, aligning with modern security approaches like Zero-Trust and the MITRE ATTK&CK framework.



Defense in Depth 

Before demonstrating this attack I will first cover the fundamentals of Defence in Depth, as this will be used later on in the project. DiD’s layered approach, which has been simplified into three macro layers, will be examined due to this attack's exploitation of the weakness in 2 critical layers. 

I will discuss Administrative controls, such as password policies, access control, and awareness training, as well as technical controls, including encryption, tools, and group policies. In this scenario I will take the perspective of an employee, as this form of attack can be performed by a current or former employee or through a compromised account.




Attack



After installing Microsoft C++ the attack is initiated on the attacker's computer using command prompt to search for a specified user and extract their password hash. This hash is then run in a program like hashcat, where it is tested against possible passwords sourced from previous data breaches or common variations - as highlighted by the nordpass common passwords report. 

This process exploits the weakness in both administrative and technical controls covered previously.  In administrative controls, the weakness we see here is weak password policies because when this is taken into hashcat the password hash is tested against a long list of likely passwords and passwords gathered in data breaches. 



	Moreover technical controls like antiquated encryption methods as we will see in the detection portion of this project are also leveraged to allow this password hash to be grabbed which allow for offline cracking, which is important as this effectively allows for the attacker to get around security measures like a lockout after too many failed login attempts.





Detection 

Since the actual password cracking occurs offline, detecting this attack relies entirely on identifying the initial ticket extraction phase. To understand this, it is helpful to look at the AD kerberos ticketing system. Kerberos is the default authentication protocol for windows domains; instead of sending passwords over the network, it uses encrypted “tickets” to grant users access to services.



In the reference photo, we can observe windows event ID 4769, which logs a Kerberos Service Ticket (TGS) request. While a single request is normal network behavior, seeing a massive volume of these requests in rapid succession is incredibly suspicious and a strong indicator of reconnaissance. 

Moreover, the information tab below the event log reveals the use of the 0x1f which likely indicates the use of RC4 encryption. RC4 is an antiquated cipher that is widely considered obsolete and insecure, making the stolen hashes relatively easy to grab, allowing the hacker to crack them offline. 
	To mitigate this vulnerability, organization must transition away from RC4 when able , and enforce modern, robust encryption standards like AES-256

To properly identify these anomalies in real-time, security teams should implement Security Information and Event management (SIEM) tools configured to alert on unusual spikes in Event ID 4769. By aligning these monitoring strategies with the MITRE ATT&CK framework (Specifically techniques like kerberoasting) and adhering to Microsoft’s official security hardening guidelines, organizations can greatly enhance their defensive posture.

Mitigation strategies

Password policies and administrative controls

To tackle this attack, we will start with the gold standard of defence: a robust password policy. First passwords should be long, at an absolute minimum,  14 characters or more. 
Short password resets are ineffective because humans will often fall back on what is convenient over what is secure - If the period between required password changes is too short, people might resort to predictable patterns like “SummerPassword2026.” 

In our administrative controls, we can force passwords to meet specific length requirements and allow special characters by configuring group policy within AD’s ‘users and computers’ menu. We can also encourage, through training, the use of passphrases, or keyword strings that make for stronger passwords, like ‘Coffee_microphone_mouse-19’.
These are much easier to remember just by scanning a desk, but are incredibly more difficult for an attacker to crack via bruit force. 
To further bolster this policy , organizations must actively ban the reuse of previous passwords and strongly recommend password managers to help users securely store complex credentials so passwords are not the same for all accounts.

Two-Factor Authentication (2FA)

Another critical way to strengthen accounts is through Two-Factor Authentication (2FA); While 2FA is not supported natively in standard AD logons, organizations can integrate options to enforce it. Implementing these changes, like Microsoft Entra ID (with conditional access policies) or Duo security ensures that even if an attacker successfully extracts and cracks a password hash offline, they cannot use these credentials without the second verification factor.

Principle of Least Privilege (PoLP)

Beyond passwords and authentication, we must heavily restrict access by enforcing the Principle of Least Privilege. 
By ensuring that users and services accounts only have the exact permissions necessary to perform their specific roles, we severely limit the “blast radius” if an account is compromised. 

This is best implemented through Role-Based Access Control (RBAC). Moreover, for highly sensitive administrative tasks, organizations should utilize Just-in-Time (JIT) privileged access. Rather than leaving standing administrative rights permanently active and vulnerable, JIT grants temporary, time-bound elevated privileges only when explicitly requested and approved.

Continuous visibility and monitoring

Finally, a strong DID strategy requires continuous log aggregation and monitoring. Organizations should utilize tools such as Microsoft defender for identity and SIEM platforms like splunk. These tools aggregate and analyze network activity in real-time, allowing security teams to quickly detect anomalous behaviors, such as unusual kerberos ticket traffic or lateral movement, and shut down the attack before a compromised account can be exploited to its fullest extent.

Conclusion



	In summary, effective cybersecurity in the face of sophisticated attacks necessitates a comprehensive Defence-in-Depth approach. 
As demonstrated, the exploitation of obsolete encryption systems and the vulnerabilities in administrative and technical controls present a significant threat. 
However, by implementing targeted mitigation strategies, including strong password policies, mandatory 2FA, and principle of least privilege controls, organizations can significantly defend their environments. 
	Continuous monitoring and tools like Microsoft defender for identity and Splunk, guided by frameworks like MITRE ATT&CK, ensures sustained vigilance and rapid responses from blue team members against an evolving cyber threat.

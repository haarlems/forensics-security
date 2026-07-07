
# thor

Seems like this is the doing of an agent of chaos.
See if you can shed some light.

Artifacts: 
`thor.pcap`

1. What is the attacker's user agent?

The flag format is `SSS{user-agent-string}`

2. What is the victim's username and hostname?

The flag format is `SSS{username-hostname}`

3. What NTLM hash did the user authenticate with?

The flag format is `SSS{hash}`

4. What is the DNS computer name?

The flag format is `SSS{dnscomputername}`

Hint 1:
what unusual user agent can you find in the pcap?

Hint 2:
visible in the data payload.

Hint 3:
why does wireshark hint to a Global NT Password?

Hint 4:
the ntmlssp.challenge.target_info exposes this information.

Solution in `../solution/`

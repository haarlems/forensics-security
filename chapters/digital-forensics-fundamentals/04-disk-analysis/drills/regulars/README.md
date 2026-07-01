
# regulars

Some more questions arise from the same set of given artifacts, see if you can find evidence to answer them all.

Artifacts: 
`uac.tar.gz`

1. What executable did the attacker run to establish persistence?

The flag format is `SSS{full/path/to/executable}`

2. What was the full cmdline they used?

The flag format is `SSS{full -cmdline here}`

3. What secondary persistence is established by the backdoor user?

The flag format is `SSS{/full/path/to/executable}`

4. What time was the secondary persistence planted by the backdoor user?

The flag format is `SSS{yyyy-MM-dd-hh:mm:ss}`

5. What is the first backdoor user the attacker logged in with and what shell does it use?

The flag format is `SSS{/home/fullusername:/full/path/to/this/user's/shell/}`

6. What time did the attacker first log in as the backdoor user?

The flag format is `SSS{yyyy-MM-dd-hh:mm}`

7. What other user did the attacker try to login with?

The flag format is `SSS{fullusername}`

Artifacts:
`uac.tar.gz`

Hint 1:
you're probably seeking the command line.

Hint 2:
what logs show the `CMDLINE` or `COMMAND` executed?

Hint 3-1:
what config did the attacker edit?

Hint 3-2:
how does a user `run commands`?

Hint 4:
what artifacts show us the timestamps on ext4?

Hint 5-1:
what log file shows us successful logins?

Hint 5-2:
what file shows user's shells?

Hint 6:
what log file shows us successful logins?

Hint 7:
what log file shows us failed logins?

Solution in `../solution/`

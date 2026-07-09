
# red

Something weird was happening on this machine, sift through the triage artifacts to find what happened.

Artifacts: 
`red.zip`

1. What is the earliest time of infection?

The flag format is `SSS{yyyy-MM-dd-hh:mm:ss}`

2. When did the victim first interact with the actual payload?

The flag format is `SSS{yyyy-MM-dd-hh:mm:ss}`

3. What was used to execute the payload?

The flag format is `SSS{AppIdDescription}`

4. What is the commandline that downloaded the second stage?

The flag format is `SSS{"full" -command line}`

5. What directory was the second stage dropped into?

The flag format is `SSS{\pa\th\}`

Hint 1:
what is the first file the attacker dropped?

Hint 2:
what file does Windows create when a user first interacts with a file?

Hint 3:
you are seeking an automatic destination to find the AppIdDescription.

Hint 4:
where can you see the Executable Info?

Hint 5:
you seek either the event to show you these details, or evidence of program execution.

Solution in `../solution/`

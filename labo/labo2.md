# Oefening 5

`cat -` schrijft waarden 

`reset` hersteld terminal

`mkfifo t` named file

`stat /etc/passwd` info file
`man 2 stat`

`strace [command]`

0 -> standaard invoer
1 -> standaard uitvoer
2 -> standaard error uitvoer

1. trace handig om te weten wat er in de achtergrond gebeurt


Zombie process kan niet worden gestopt.
Als parent process nie luistert naar de exit status van zijn kinderen blijft het parent process aanmaken

bij creatie van een nieuw kind process (word gedaan met fork), krijgt als return waarde in parent het pid, child krijgt waarde 0 terug. zo kan herkent worden welke type het process is.

`ps -ef|less` tty --> ? is deamon
`<defunct>` slecht werkend process wss zombie

`top` heeft counter met zombies

`htop` geavanceerdere top

een zombie krijg je als een child process sneller klaar is dan een parent process
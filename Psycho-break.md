I started the room by enumerating the target machine and checking the web server.
I used Gobuster to find hidden directories and files:

gobuster dir -u http://machine_ip -w /usr/share/wordlists/dirb/common.txt

I found some interesting files like index.php and map.php. I also tried using a bigger wordlist and extensions:

gobuster dir -u http://machine_ip -w /usr/share/wordlists/dirb/big.txt -x php,txt,html

The interesting results included:

/index.php
/map.php
/css/
/js/

I checked the source of the pages using curl:

curl -s http://machine_ip/index.php

and:

curl -s http://machine_ip/map.php

The important clue I found was:

Escape from Laura to find it

The room also mentioned hidden pages/files such as you_made_it.txt, so I continued enumerating the web application. The Gobuster enumeration and the room hints pointed me towards the hidden paths rather than the normal homepage.

Finding the hidden file

I checked the discovered files and eventually reached the part of the challenge involving the image/audio files.

I initially thought that Table.jpg might contain steganography, so I checked it with:

exiftool Table.jpg

The interesting thing was that Table.jpg was actually detected as a ZIP file, and it contained:

Joseph_Oda.jpg

So I extracted it:

unzip Table.jpg

The ExifTool output showed the ZIP structure and the embedded Joseph_Oda.jpg.

After extracting it, I also got:

key.wav

The room was hiding information in the audio, so I generated a spectrogram:

sox key.wav -n spectrogram -o key_spectrogram.png

I also tried increasing the contrast:

sox key.wav -n spectrogram -z 120 -o key_spectrogram_z120.png

The room notes indicate that the hidden message was intended to be extracted from the audio spectrogram.

FTP

I then checked the FTP service and found files belonging to the joseph user.

The directory listing showed:

program
random.dic

So I downloaded both:

get program
get random.dic

The FTP listing confirmed that both files were available.

I checked the program:

./program

and got:

[+] Usage


./program <word>

So I understood that random.dic was supposed to be used as a list of possible words for the program.

I initially tried:

./program < random.dic

but that only gave the usage message because the program expects the word as an argument, not through stdin.

So I used a loop to test every word:

while read -r w; do
    ./program "$w"
done < random.dic

After filtering the dictionary based on the required format, I eventually found:

kidman

The program/decryption stage then gave me:

kidmanpasswordissostrange

I got the SSH.

Then i wanted user.txt and root.txt
i got user.txt easily in a directory ahead the root directory.

For root.txt , i inspected cronjobs and found:

*/2 * * * * root python3 /var/.the_eye_of_ruvik.py

this file was getting executed every 1/2 min as root and has an edit permission on , so we injected our payload in it and started a listener on the attackbox and got a shell in under a minute.

after getting the shell , then got root flag in /root/root.txt

Final Task was to delete ruvik's account so i again could have used cronjobs but why make the process longer. i directly used the following command in the shell i got :

userdel -r ruvik

and BOOM , ruvik was gone hahaha


Download all of the things!
===========================

This is a short set of notes for various commands useful for downloading and uploading different things.

Scripting language test servers
-------------------------------

First up, we have test servers. These are useful for temporarily hosting content on a LAN or for an engagement. They start simple web servers with the webroot being the working directory from which the server was run.

Python 3:

```
python3 -m http.server <optional:port>

```

Port is optional; by default it serves on port 8000, but this can be changed if needed by specifying.

PHP:

```
php -S <address>:<port>

```

This gives you a PHP enabled web server. This can be useful for internal testing or perhaps spoofing. Usually, I'll use this with either localhost, or 0.0.0.0. Localhost is *not* availible to other networked devices, while 0.0.0.0 is. E.g. if I'm wanting to test a PHP page on my phone while serving from my laptop, I'd use 0.0.0.0 or so. The port is arbitrary.

Downloading things
------------------

The following apps let you download things:

#### Wget

Linux command line app for fetching pages. Basic use is

```
wget <URL> 

```

Simple as that. For mass downloads, e.g. like if you're trying to rip someone's book library or something, you could do:

```
wget --no-parent --recursive <URL>

```

To get more complicated, you can filter results with regular expressions, e.g. for when you only want to pull one type of file, etc:

```
wget -np -r --accept-regex pdf <URL>

```

This would make it so that only files containing "pdf" in their file name would be downloaded. -np and -r are simply short forms for --no-parent and -recursive.

Wget is also a commandlet alias in powershell for the invoke-webrequest commandlet, so similar syntax can be used in powershell for downloads as well. Note though that you have to specify the output file on powershell, so you'd tack on -O &lt;filename&gt; in order to download. I'm unsure if the more interesting options like --accept-regex, etc, are availible in powershell, though.

#### Youtube-dl

Youtube-dl is a downloader that can rip videos from Youtube and similar sites. Good for archiving videos and ripping music.

Basic use to download a video:

```
youtube-dl <URL to video>

```

Basic use for ripping a video's audio:

```
youtube-dl -x --audio-format <an audio format> <URL to video>

```

Note that if you point it at a playlist URL, it can download the entire playlist. Has trouble with age restricted videos, though. There's a way to make it authenticate into an account to get these to work, but I haven't done this before.

#### Certutil

Certutil is a Windows component that is used to fetch certificates, but by abusing it, we can use it as a simple download tool when no other method is availible. E.g. when we only have access to CMD, etc:

```
certutil.exe -urlcache -split -f <URL to remote file> <what to save the file as>

```

More applications like this can be found on the [LOLBAS GitHub](https://lolbas-project.github.io/#).

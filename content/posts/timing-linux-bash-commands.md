---
title: "Timing Linux Bash Commands"
date: 2017-03-26T15:46:03-06:00
draft: false
type: "post"
tags:
  - bash
  - linux
  - time
aliases:
  - /blog/timing-linux-bash-commands
---

While updating some [Vagrant](https://vagrantup.com) config files, I was curious how long it was taking for a box to build from scratch. I tried dumping out timestamps directly in the `Vagrantfile`, but ultimately decided to look for a Bash-specific method that would be more universal and let me time some other Linux commands. Here's what I came up with.

Open your `.bashrc` in your favorite text editor.

```
$ vim ~/.bashrc
```

Add this function to the end of the file

```
function how-long-is()
{
    SECONDS=0
    "$@"
    duration=$SECONDS
    echo "$(($duration / 60)) minutes and $(($duration % 60)) seconds elapsed."
}
```

Save the file and return to the command line. Reload your `.bashrc` file.

```
$ source ~/.bashrc
```

Now you can test out the new function.

```
$ how-long-is sleep 2
```

You should see output similar to:

```
$ how-long-is sleep 2
0 minutes and 2 seconds elapsed.
```

Enjoy timing your Bash commands! :)

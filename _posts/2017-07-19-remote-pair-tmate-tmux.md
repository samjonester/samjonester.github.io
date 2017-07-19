---
layout: post
title: "Remote Pairing with Tmate and Tmux"
description: "Have an existing tmux session? Need to pair remotely? Share it with tmate!"
date: 2017-07-19
tags:
 - tmux
comments: false
share: true
---

## Here's the situation.

You LOVE [tmux][tmux], and you use it all the time to organize your terminal and create a workspace. You've created a new tmux session where you will do your work named `funproject`.

``` sh
tmux new-session -s funproject
```

Then you've created a bunch of windows, splits, and running processes, and **now you would like to pair**.

## Possible solutions

💔 **Jump Box**
You could [set up a jump box][remote-pairing-ssh] to allow a remote pair to ssh directly into your machine. This uses ssh to create a reverse tunnel to the public jumpbox, exposing your ssh port to your pair, who connects with an ssh key. This solution is secure, but it's complicated to set up. You'll also need to pay for a public jump box on Digital Ocean (or similar cloud service).

💚 **Tmate**
[Tmate][tmate] does all of that jump box work for you, and it's REALLY easy to use. It's a fork of tmux that allows you to share a terminal work session via a shareable ssh key. It even provides a read-only ssh key if you'd like to provide that instead. It isn't quite as secure because a key is not required to connect, but it sure is convenient! Just be careful to pair responsibly, with those you trust giving access to your computer. Also don't forget to shut down the tmate session when you're done to close off unsupervised access to all your stuff.


-------------

## Sharing with tmate

When you first start tmate, you'll notice that it creates a new tmux session for you to share your work. You'll also frustratingly find that since it's a fork of tmux, you're unable to start sharing by attaching to your existing `funproject`.

Luckily, there is a solution! Tmate can be used strictly as the sharing vessel, and you can connect to that tmux session inside the sharing vessel with one handy command `TMUX='' tmux attach-session -t funproject`. This command allows you to nest a tmux session inside tmate.

Cool 😎! Now, let's clean up those status bars and do some scripting to make them play nicely together.

### Configure tmate to be invisible

Tmate allows a configuration file to overlay your `~/.tmux.conf` file. You can do this by creating the following file at `~/.tmate.conf`. This example will set up tmate to be as non-intrusive as possible, allowing you to focus on the tmux session you're working in.

```
## tmate

# Reassign prefix to not conflict with tmux
set -g prefix C-]
bind-key ] send-prefix

# turn off status bar so tmate is invisible
set -g status off

# Fix timeout for escape key
set -s escape-time 0
```

### Scripting the Integration
The following functions script the integration between tmate and tmux. This allows a new tmate session to be started and stopped with a simple function. This function can load an existing tmux session for sharing. To do this, add the following functions to your `~/.bashrc`.

``` sh
# TMATE Functions

TMATE_PAIR_NAME="$(whoami)-pair"
TMATE_SOCKET_LOCATION="/tmp/tmate-pair.sock"
TMATE_TMUX_SESSION="/tmp/tmate-tmux-session"

# Get current tmate connection url
tmate-url() {
  url="$(tmate -S $TMATE_SOCKET_LOCATION display -p '#{tmate_ssh}')"
  echo "$url" | tr -d '\n' | pbcopy
  echo "Copied tmate url for $TMATE_PAIR_NAME:"
  echo "$url"
}



# Start a new tmate pair session if one doesn't already exist
# If creating a new session, the first argument can be an existing TMUX session to connect to automatically
tmate-pair() {
  if [ ! -e "$TMATE_SOCKET_LOCATION" ]; then
    tmate -S "$TMATE_SOCKET_LOCATION" -f "$HOME/.tmate.conf" new-session -d -s "$TMATE_PAIR_NAME"

    while [ -z "$url" ]; do
      url="$(tmate -S $TMATE_SOCKET_LOCATION display -p '#{tmate_ssh}')"
    done
    tmate-url
    sleep 1

    if [ -n "$1" ]; then
      echo $1 > $TMATE_TMUX_SESSION
      tmate -S "$TMATE_SOCKET_LOCATION" send -t "$TMATE_PAIR_NAME" "TMUX='' tmux attach-session -t $1" ENTER
    fi
  fi
  tmate -S "$TMATE_SOCKET_LOCATION" attach-session -t "$TMATE_PAIR_NAME"
}



# Close the pair because security
tmate-unpair() {
  if [ -e "$TMATE_SOCKET_LOCATION" ]; then
    if [ -e "$TMATE_SOCKET_LOCATION" ]; then
      tmux detach -s $(cat $TMATE_TMUX_SESSION)
      rm -f $TMATE_TMUX_SESSION
    fi

    tmate -S "$TMATE_SOCKET_LOCATION" kill-session -t "$TMATE_PAIR_NAME"
    echo "Killed session $TMATE_PAIR_NAME"
  else
    echo "Session already killed"
  fi
}
```

### Sharing with tmate

Open a pair session and attach to your existing tmux session. The tmate share url is printed and copied to your (OS X) clipboard.

``` sh
tmate-pair foo
```

Open a pair session with a blank terminal.

``` sh
tmate-pair
```

### Ending a pair

After you're done, close down tmate. This will also detach all clients from the shared tmux session if one was provided when starting the pair. Remember to do this to close off unsupervised access to all your stuff!

``` sh
tmate-unpair
```

[remote-pairing-ssh]: http://www.zeespencer.com/building-a-remote-pairing-setup/
[tmux]: https://github.com/tmux/tmux/wiki
[tmate]: https://tmate.io/


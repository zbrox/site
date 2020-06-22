+++
title = "Nix package manager on macOS 10.15 a.k.a Catalina"
date = 2020-05-15T20:03:13+02:00
slug = "nix-on-macos-catalina"

[taxonomies]
tags = ["nix", "macos"]
+++

Trying to install Nix on macOS Catalina is a tiny bit more difficult than I imagined. I had completely forgotten that Catalina doesn't allow you to touch the root filesystem and the Nix store is at the usual place - `/nix`. When trying to install Nix, you'll get an error that `/nix` is not writeable. The way around that is to create a so-called synthetic firmlink (a feature of APFS) which would allow us to mount volumes on the root filesystem. Those volumes are created only on boot so after we define one, we, unfortunately, have to restart the computer.

How do we create those synthetic firmlinks? If you don't already have the `/etc/synthetic.conf` file, create it with the appropriate permissions.

```sh
sudo touch /etc/synthetic.conf
sudo chmod 644 /etc/synthetic.conf
```

In that file add a line with just the text `nix` in there which tells the system to have a firmlink at `/nix` on boot. We will later define what's gonna be mounted there. But first, time to restart the computer. Yay!

And here's how to do that. If we assume you have not partitioned your Mac's disk or did something other than just installed macOS, so `disk1` is your main drive. First, we add an APFS volume with the `/nix` mount point with a volume name of `Nix`. The volume name you can change to whatever you want.

```sh
sudo diskutil apfs addVolume disk1 APFSX Nix -mountpoint /nix
```

Then we need to do a couple of more things like enable ownership on the volume (from the man pages of `diskutil`: __When ownership is enabled, the Owner and Group ID settings that exist on the disk are taken into account for determining access, and exact settings are written to the disk as FSOs are created.__), hide the volume from appearing on the Desktop, and add an `fstab` entry so it automounts on restart.

```sh
sudo diskutil enableOwnership /nix # enables ownership
sudo chflags hidden /nix  # hides the volume from the Desktop, still shows up on the Finder sidebar
echo "LABEL=Nix /nix apfs rw" | sudo tee -a /etc/fstab # add an entry to fstab to automount
```

After this you can install Nix the usual way and everything should be fine. Ta-da!
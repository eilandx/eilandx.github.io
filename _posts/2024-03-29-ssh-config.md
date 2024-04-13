---
title: Configuring your SSH config file
date: 2024-03-29 
categories: [Guide, Tutorial ] 
tags: [ssh, workflow, efficiency, beginner-friendly]    
author: CrimsonCloak 
---
<!-- Add little information for preview snippet -->
In this guide, we will show you how to properly set up an SSH config file for your user. Doing so will enable you to improve your workflow and avoid using long SSH-commands repetitively and help you avoid potential typo's or mistakes. 

# Why is an SSH config file useful?

Imagine you are managing multiple devices or need to perform maintenance tasks on different servers. Every time you would open up a remote SSH-connection, you most likely use an SSH command such as `ssh [USERNAME]@[IP]`, followed by inputting your user's password. More advanced users maybe even strictly use ssh-keys to connect to their devices. In that case, your command becomes something like `ssh -i [/path/to/private_key] [USERNAME]@[IP]`, adding your IdentityFile (~ private key file) location to your command.

You can already tell that if you're switching between these servers or devices frequently, it can become quite tedious to keep typing these long commands, even if you're making use of your shell's history. An SSH-configuration file can help you manage this so this becomes more efficient, less error-prone and less annoying in general!

## Defining your SSH configuration file

### Location

In order to make use of an SSH configuration file, we need to create a new file inside your user's .ssh directory - which is usually located under your user's home directory. For example, if I'm logged in as user "crimsoncloak" on a Linux OS, my .ssh directory would be located at `/home/crimsoncloak/.ssh`. For a Windows machine, this would be located at `"C:\Users\crimsoncloak\.ssh"` (be mindful of the difference between backslashes and forward slashes!).

Inside of that `.ssh` directory, we create a new file called `config`, without any file extension. This is the file in which we will define all of our SSH configurations that will be picked up when we use `ssh` or `scp` commands with our specific user. 

### Content

When we create this file, it will obviously start out empty. We can then start populating it with different configurations for specific servers, devices or use cases for our ssh-usage. The configuration file can contain a great deal of options, but we will limit this user guide to the basics. A more in-depth overview of all the possibilities can be found in the man pages or using online sources.


### Example configuration file

For a very basic example, let's say we have a server or VM with the local IP address 192.168.10.10 and we want to connect to it as the 'admin' user with the very secure password 'changeme'. If you want to define this in your configuration, you can add the following lines to your `~/.ssh/config` file:

```
Host myserver
  Hostname 192.168.10.10
  User admin
  PasswordAuthentication yes
```

By defining this configuration, whenever you execute `ssh myserver`, your machine will attempt to connect to the IP 192.168.10.10 with the user 'admin' and prompt you for your password, as you would normally when connecting to a server. 

Note that now, the behavior of `ssh myserver` is now exactly the same as the command `ssh admin@192.168.10.10`! You can define as many Hosts as you want in your configuration, and you can even play around with more advanced use like using jump hosts, different ports and so much more!

## Example use cases

Below we will list some common or interesting use cases you might encounter. Please note that these are just a few of the possible configurations you can define! None of these configurations will match your specific use cases, so be sure to adjust their contents to what you need for your remote connections. They can give a good indication of what is possible with the SSH configuration file, however! 

As this guide is mainly aimed at beginners, you will see password authentication being used a lot. We wholeheartedly recommend using SSH-keys for your connections, as they are more secure to use and more usable in the long run. For testing out some SSH-configurations, however, they can be a great starting point!


### Defining multiple servers
If you have multiple servers you connect to often, a possible configuration could look like this:

```
Host myserver
  Hostname 192.168.10.10
  User admin
  PasswordAuthentication yes

Host myserver2
  Hostname 192.168.10.11
  User admin
  PasswordAuthentication yes
```
With this setup, you can use both `ssh myserver` and `ssh myserver2` to connect to your specific servers!

### Connecting to a server with a specific user and port
If you wish to connect to a server that uses a a different port than 22, you can also define this in your config file as such:

```
Host myserver
  Hostname 192.168.10.10
  User admin
  Port 2222
  PasswordAuthentication yes
```
### Connecting to a server with a specific user and private key file
Using SSH-keys is an obvious (security) improvement over using password authentication. If you have a private key file that you use to connect to a server, you can define this in your configuration file as such:

```
Host myserver
  Hostname 192.168.10.10
  User admin
  IdentityFile ~/.ssh/id_rsa
```

In the example above, we assume your private key file is located at `~/.ssh/id_rsa` and the corresponding public key has been added to your user's `authorized_keys` file. This way, you can connect to your server using `ssh myserver` without having to specify the private key file every time. Do note that if you move your private key, you will need to update your `config` file accordingly!

## Conclusion

When you want to make your life easier managing remote SSH-connections, your SSH configuration file can be a great asset to your workflow. By defining your SSH-commands inside your `config` file, you don't have to retype your long commands or go through your shell's history to find your specific command. Using the configuration file can save you a lot of time, improve your workflow and make your experience switching remote hosts a lot more pleasant. 





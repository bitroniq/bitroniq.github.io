https://wiki.gentoo.org/wiki/SSH_jump_host This page was last edited on 10 January 2019, at 18:02.
Dynamic jumphost list
You can use the -J option to jump through a host:

user $ssh -J host1 host2
If usernames or ports on machines differ, specify them:

user $ssh -J user1@host1:port1 user2@host2:port2
Multiple jumps
The same syntax can be used to make jumps over multiple machines:

user $ssh -J user1@host1:port1,user2@host2:port2 user3@host3

Static jumphost list
Static jumphost list means, that you know the jumphost or jumphosts you need, to reach a host. Therefore you can create a static jumphost 'routing' in ~/.ssh/config file. The advantage in comparison to the dynamic jumphost option is, that you don't have to provide the .ssh config on jumphosts between your machine and all the other jumphosts between you and the final host you want to jump to.

Setup
FILE ~/.ssh/configProxyJump Example
### First jumphost. Directly reachable
Host betajump
  HostName jumphost1.example.org
 
### Host to jump to via jumphost1.example.org
Host behindbeta
  HostName behindbeta.example.org
  ProxyJump  betajump
  
# Old way: https://wiki.centos.org/TipsAndTricks/SshTips/JumpHost

# New way: https://www.madboa.com/blog/2017/11/02/ssh-proxyjump/
The Old Way
I’ve used the ProxyCommand for some time now, relying on nc to push SSH traffic over an established tunnel. Without going into the gory details, the process boils down to

setting up an SSH session using the -D option to establish a SOCKS5 port-forwarding connection,
configuring SSH to use a ProxyCommand to push traffic through the SOCKS5 connection.
It works reasonably well if you have a decent version of nc and you’ll be using that SOCKS5 tunnel for several connections. You can also use the SOCKS connection with web browsers to reach remote-internal web servers.

The New Way
Sometimes, however, you may want to avoid the two-step process, or you may be on a host that doesn’t have all the tools you need for SOCKS connections.

The new -J (aka ProxyJump) command is tailor-made for you! Here’s the basic invocation:

ssh -J your.jump.host remote.internal.host
You’ll end up logged into the remote internal host, and ssh automatically takes care of the intermediate step of logging into the jump host first.

You can even use it as an option for secure file copies:

scp -o 'ProxyJump your.jump.host' myfile.txt remote.internal.host:/my/dir
The file myfile.txt will end up in the /my/dir directory on your remote internal host.
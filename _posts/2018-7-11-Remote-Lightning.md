---
layout: post
title: Connecting to a Remote Lightning Node
---

This post describes how to make your local lightning node connect to a remote node. 
First set up an ssh tunnel between you and your remote and make it listen to bitcoin's 
default rpc port: 

    ssh -i /path/to/your/ssh_key.pem -f <USER>@<SERVER_IP> -L 8332:127.0.0.1:8332 -N

If you authenticate to ssh with a password, you can leave out the `-i` option. 
There are two different ways at the moment to authenticate to bitcoind. One is the standard
rpcuser/rpcpassword method. This however is about to be deprecated. It is recommend to use the
new 'cookie' authentication. For this first generate a token, bitcoin provides a tool for this 
in the bitcoin core repository. It is located in share/rpcauth. To get a new token for the user
'rpcuser' run `./rpcauth.py rpcuser`. The output will be similar to:

    rpcauth=rpcuser:fac3ce16b0855ef141398ac69b1f340$19524bd5d63262d8928385e5d8a694e3d0ec98046b540a1faa7d016c3622d8d3                                                                                                                                                                                     
    Your password:                                                                                                                                                                                                                                                                                       
    tvIZqauaRAJ1oU1gDS0JSRgPmR1FEzJ6bfM2CKxfy_4=

Now open your `bitcoin.conf` file in your bitcoin data directory and add the following lines. 
You can add as many different rpcauth lines with different users as you want to. I added the
`#` commented line for the password to make sure I can lookup the password again at a later stage,
however it can be left out. Ideally it should be stored only on the client side:

    txindex=1                                                                                                                                                                                                                                                                                            
    testnet=1                                                                                                                                                                                                                                                                                            
    whitelist=0.0.0.0/0                                                                                                                                                                                                                                                                                  
    server=1                                                                                                                                                                                                                                                                                             
    rpcauth=rpcuser:fac3ce16b0855ef141398ac69b1f340$19524bd5d63262d8928385e5d8a694e3d0ec98046b540a1faa7d016c3622d8d3                                                                                                                                                                                                                                                                                 
    # password = tvIZqauaRAJ1oU1gDS0JSRgPmR1FEzJ6bfM2CKxfy_4=
    rpcport=8332 
    
    
Leave out the `testnet=1` to run it on mainnet. Save and close the file, then start `bitcoind` on your remote host. 
You can track its syncing progress by running `tail -f debug.log` in your remote host bitcoin data dir. 

Now install `bitcoin-cli` on your local machine. To test if the ssh tunnel is running correctly execute the following 
line on your local machine:

    bitcoin-cli -rpcuser=rpcuser -rpcpassword=tvIZqauaRAJ1oU1gDS0JSRgPmR1FEzJ6bfM2CKxfy_4= -rpcport=8332 getblockchaininfo

Once the remote machine is synced start your local lightning daemon with:

    lightningd/lightningd --bitcoin-rpcuser=rpcuser --bitcoin-rpcpassword=tvIZqauaRAJ1oU1gDS0JSRgPmR1FEzJ6bfM2CKxfy_4=
    --bitcoin-rpcport=8332 --network=testnet --log-level=debug

Replace `--network=testnet` with `--network=bitcoin` to run mainnet.  When restarting your local machine, 
you need to re-open the ssh tunnel again.

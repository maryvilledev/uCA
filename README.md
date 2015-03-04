# uCA
uCA is a micro-CA that uses OpenSSL to allow you to easily create signed certificates with multiple SubjectAltNames

## Why

Have you ever tried to sign certificates with SubjectAltNames using OpenSSL's CA?  It's a gigantic pain.

All I wanted to do was create my own little internal PKI, so my apps could do mutual-auth SSL with each other.
They needed to trust one another but not necessarily anything else.  
However, to make things easy to scale, I wanted to make it so that I could have, for instance, rmq1, rmq2, and rmq3, and all of those would present a cert that *also* said the host was rmq.
Then I could just use haproxy or something in TCP mode and forward connections to whichever backend I chose, hit it as "rmq", and have everything work nicely.

Shoulda been easy.

Wasn't.

Almost all of the extant OpenSSL documentation that talks about how to do SANs assumes you don't mind editing openssl.cnf in between every single time you generate a certificate.
Sure, I could have automated that too with some template markers and a loop around it, but that felt really gross.

### Why not ditch OpenSSL?

Everything else is even worse.

The small things, like xCA, either don't do SANs at all, or they require some stupid interactive X client to work, or both.  I wanted something I could trivially script.

Dogtag is cool, but it proved to be too hard for me to separate from the rest of the Fedora machinery, and I don't want Fedora.

EJBCA and OpenCA are way, *way*, **way** too big and complex and featureful for what I wanted to do.

## What

So I dug through a whole bunch of conflicting web pages, and played with how OpenSSL interacts with the environment, and eventually I came up with a set of recipes that use a grotesque and finicky dance between openssl.cnf and its environment to let you programmatically generate SANs for your certs and sign 'em.
Then you can use them as client certificates and have everything work more-or-less cheerfully.

And now, by using uCA, *you* don't have to bang *your* head on those particular bricks anymore!

## How

First thing, copy openssl.cnf.template to openssl.cnf, and edit the organization stuff (lines 133-160) to reflect your use case.  Unless, of course, you do want to be the Garden Weasel Attack Squad from Cuba, Missouri. 

Next, start running the things in scripts.

* If you want to build a CA and keep it around for a while, use createCA.  If you want a particular passphrase, stick it in CA_PASSPHRASE in the environment; if you don't a random passphrase will be generated.
* Then run newcert name [ **server** | client | both ] # (server is the default)
* If on the other hand you want to do all your certs at once, use build_collection and feed it a list of cert names.  You may want to change "both" (for the cert usage) in that script.
* Currently, SAN generation is controlled in the function build_subj_and_san in uCA-utils.sh ; edit this to change the rules I'm using (which are to make anything with trailing digits also accept the same name with no digits).  This function is likely to get broken out into its own file soon.

I hope you find uCA as useful as I have!
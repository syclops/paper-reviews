# CONIKS: Bringing key transparency to end users

## Metadata

Authors: Marcela S Melara, Aaron Blankstein, Joseph Bonneau, Edward W Felten,
Michael J Freedman

Year: 2015

Venue: USENIX Security

## Summary

CONIKS is a system to bring better consistency, privacy, and bandwidth
efficiency to end-user public key directories. A key directory should not be
thought of as a CA but rather as a provider of authenticated name-key bindings
for its own users (such as for private messaging). In this scenario, it is
important to ensure that all entities that query the provider for the keys of a
specific user observe the same information while also allowing providers to hide
their users' names and public keys.

Identity providers in CONIKS store their users' name-key bindings in the form of
a Merkle prefix tree (a binary tree in which each non-root node represents a
binary prefix, and each node contains a hash derived from information stored at
that node or the hash of its child nodes). Providers can use a verifiable
unpredictable function (VUF) to assign each username to a prefix such that a
prefix does not leak any information about other names in the directory.
Providers also store a commitment to a user's public keys rather than the public
keys themselves to avoid leaking information about a given user's public keys
(in particular, if someone tries to repeatedly test for public keys associated
with a known prefix).

Clients can then look up names at identity providers, who provide the key
information (if any) along with the proofs required to verify this information.
Clients can also query their own names to ensure that the provider is not
tampering with the user information. Auditors (who can be providers, clients, or
any entity in the system) verify and exchange provider-signed roots of the
Merkle prefix tree in order to detect equivocation. The roots of a provider's
Merkle prefix trees are also chained in time to prevent forked histories.

Clients can have different policies that enforce defaults on either side of the
security-usability tradeoff. The default is on the side of usability, but a
stricter policy requires all key changes to be signed by the old key and also
encrypts the public key data so that public keys can be selectively shared.
These policies have different implications for what a malicious provider can do
and how a user can react to the loss or compromise of an associated private key.

The authors provide a secure chat application based on CONIKS and OTR, and show
that the bandwidth cost of querying, monitoring, and auditing identity providers
is low (on the order of 10 kB/day with 10 million users).

## Review

The paper addresses an interesting and underexplored problem. Many recent papers
in the PKI space focus on the TLS PKI, where the namespace is simply the global
DNS namespace and thus public information. At the smaller provider level,
namespaces are local and there is a clear need to keep these private in some
situations (and indeed, there have been some calls for this in the TLS PKI,
e.g., with the name redaction proposal in Certificate Transparency). The main
contribution of the paper lies in the mechanism to ensure the privacy of these
directories while allowing the usual querying and verification to be done as
they are in Certificate Transparency.

While the paper addresses many of the usual details for how CONIKS works in
practice, there are some areas in which it could have been clearer. No precise
protocol is given for registration, lookup, monitoring, or auditing, even though
these seem to be crucial for the use of CONIKS. One way the paper claims to
achieve efficiency is to claim that only the most recent signed root of a
provider's Merkle tree needs to be stored, but does not state how a client who
misses multiple roots in succession can then retrieve these to verify all of the
updates in between. There seems to be an attempt to mitigate this (specifically
if a provider goes offline for some time) by allowing providers to list their
next expected update, but clearly servers cannot predict their downtime in
advance.

The paper also does not address some of the hairier problems that arise in
operating a PKI, though it does acknowledge the need to address some of these
problems in future work. Most notably, it provides no specific protocol to
report misbehavior (other than "tell everyone you can") or to gossip the signed
Merkle tree roots (other than "send them to random providers"). The paper also
seems to assume that there is some list of all valid identity providers (or
specifically, a PKI that certifies these providers).

The paper also claims that more paranoid users can hide their public key data
even from those who know their prefix in a provider's Merkle tree by encrypting
the information with a symmetric key that can then be distributed to others. But
surely the entire point of having a key directory is to look up users who the
client has not previously contacted, and in this case, how can the user obtain
this symmetric key? Not to mention that anyone with this symmetric key can
distribute it to the rest of the world without leaving a clear trail of
accountability.

It is thus likely that the efficiency claimed by the paper cannot be enjoyed by
all clients, and that problems will exist for users in practice, even those that
attempt to leverage a higher level of security. However, overall the paper
provides a solid step towards a privacy-preserving key directory system.

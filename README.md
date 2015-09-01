# hshca: Hostname safe HTTP content addressing

## Summary

This is a document describing **hshca**, a specification for hostname-safe HTTP content addressing using cryptographic hashes.

It can be described in a sentence: A [multihash](https://github.com/jbenet/multihash), encoded in Base32 [RFC 4648](https://tools.ietf.org/html/rfc4648), downcased and stripped of its padding, which can be no longer than 63 bytes.

This makes it comply with [RFC 1035](http://tools.ietf.org/html/rfc1035) "labels" (more commonly known as hostnames or subdomains).

It is intended for use with content addressing over HTTP. This makes it useful for scenarios where you need to maintain the browser security model for isolating content (such as when using content addressing for serving web sites), or when your web site requires a root for relative links to work, or when you want to send content requests to different servers.

## Example

In ruby, using the [base32](https://github.com/stesla/base32) and [ruby-multihash](https://github.com/neocities/ruby-multihash) gems:

    # gem install base32 multihashes
    require 'digest'
    require 'base32'
    require 'multihashes'

    digest = Digest::SHA256.digest 'Hello World!'
    multihash = Multihashes.encode digest, 'sha2-256'

    hshca_hash = Base32.encode(multihash).gsub('=', '').downcase

    # => "ciqh7a5rmv77d7ctxew4dakiuhlf37bnjmp2hvtxfbfn3uqacjwza2i"

    Base32.decode(hshca_hash.upcase) == multihash # => true

This works for SHA256 digests (and SHA256 multihashes), but cannot support SHA512 due to the size limit (at least not according to the spec, but some implementations do ignore the RFC1035 63 octet/byte restriction).

RFC1035 originally did not allow hostnames to start with digits, but this restriction has since been relaxed with [RFC1123](https://tools.ietf.org/html/rfc1123), allowing for this spec to be standards conformant even if the encoding begins with a digit.

## Rationale

HTTP is designed for location addressing. When you visit a site (https://mysite.example.org/cat.png), first the IP address is resolved using DNS, and then the data is requested using the path (/cat.png).

Increasingly, protocols such as [IPFS](http://ipfs.io) are being designed to be **content addressable**. That is, you request the web site (or any other content) by asking for the cryptographic hash of the data you are requesting, rather than a named, location-specific resource you cannot verify.

This is useful because it enables the ability to ask any HTTP server for content, and be able to use the hash to verify that the data received is what you were expecting to receive. This potentially allows for trustless serving of web sites using HTTP. It can also conceivably be used to retrieve the same asset from multiple HTTP servers simultaneously (but *don't*, use [IPFS](http://ipfs.io) directly since it's designed for it).

In the future, browsers will support IPFS content addressing directly, and not require HTTP at all, making this spec (and probably HTTP) obsolete. In the interim, this spec allow legacy HTTP web browsers to do content addressing that works with the current domain security model. As such, this is essentially a workaround to allow for the present browser security encapsulation model while still working with content addressing, until web browsers finally integrate support for IPFS and we stop pretending HTTP is the future of the web.

## Author

This specification was written by Kyle Drake for [Neocities](https://neocities.org) usage with [IPFS](http://ipfs.io), mainly for our IPFS site archiving system.

Thanks to Juan Benet, Jeromy Johnson and the IPFS team for feedback and steering me in the base32 direction.

## Implementations

None yet!

## Pronounciation

Hush ka!

## Improvements/ideas welcome

File an issue, or send a pull request.

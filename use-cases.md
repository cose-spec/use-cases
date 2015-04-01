This page is meant to collect use case information for the COSE spec.
+Please raise issues or fork and submit pull requests, and/or send mail to [cose@ietf.org](mailto:cose@ietf.org?Subject=Use%20Cases).

# Use case: draft-selander-ace-object-security (Göran Selander)

One motivation for a compact secure data object format is end-to-end security in constrained environments in the presence of intermediary nodes, including proxies doing store-and-forward, caching of requests and responses, protocol translation (e.g. HTTP/CoAP) etc. Another setting is publish-subscribe with an intermediary broker. Another motivation is to support constrained settings where DTLS does not perform well.

Examples of information that needs to be securely wrapped include:

 *   Authorization information (e.g. list of (encoded) permissions or attributes of an associated client)
 *    Transaction keys (e.g. key of client provided to server or v.v., from a trusted third party, used to verify/decrypt data object)
 *   Resource representations (e.g. measurement values of sensors or actuator settings)
 *   CoAP messages, including payload but also CoAP Header Fields and Options, and verifying binding between request and response

The three first deals with protection of payload/application of CoAP or other protocol. The last is the protection of the CoAP message in itself. Security requirements addressing all examples include:

   * The payload shall be integrity protected and should be encrypted end-to-end from sender to receiver.

   * It shall be possible for an intended receiver to detect if it has received this message previously, i.e. replay protection.

Security requirements for protection of CoAP messages also include:

   * A Client shall be able to verify that a received CoAP message is the response to a particular request the Client made.


Optimize for the most constrained

   * The message format shall be optimized for the most constrained devices, but extensible to support a variety of use cases. For a benchmark of an optimized, but non-generic message format see Appendix C [1].

Header parameters

   * A small set of mandatory header parameters. (In [1], we propose to use essentially three parameters: cipher suite, key identifier, and sequence number. To reduce header size we assumed an enumeration of cipher suites and variable size parameters of key identifier and sequence numbers, see Appendix C [1])


CoAP message protection

   * CoAP message protection (4. above) assumes a point-to-point challenge-response message exchange supporting encryption, and integrity protection, and replay protection of request.

   * The working assumption is that in this case it is sufficient to support AEAD. Necessary keys are assumed to be established beforehand (potentially using 2. above)

   * Implicit IV. Assuming IV is composed of nonce and sequence number, for certain applications it may be assumed that the IV is established through other channel and need not be sent with each message. For generality it should be possible to include IV.

   * Algorithms with truncated MAC needs to be supported. AES-128-CCM-8 is default in CoAP.

   * For binding of request to response, a unique transaction identifier of the request needs to be integrity protected in the response. (To reduce message size, we did not include the transaction of the request in the response, only the MAC of the response is calculated over the transaction identifier, see Excluded Authentication Data [1].)


Protection of resource representation

   * For protection of resource representations (3. above) this may be point-to-point or point-to-multipoint. Example of the latter include a proxy which caches responses and sends a cached response to a matching request to reduce load on the origin server. Alternatively, a publish-subscribe setting where one publication is received by multiple subscribers.

   * For point-to-point working assumption is that in this case it is sufficient to support AEAD.
   * For certain point-to-multipoint scenarios with trusted peers, symmetric algoritms may be sufficient. In the general case, this requires a symmetric key encryption combined with private key signature of the origin server/publisher.  NIST P-256 is default in CoAP.

   * To reduce size of signature, signature algorithms with message recovery should be considered. (We should engage CFRG in this question.)


Protection of transaction keys and authorization information

   * Key wrap shall be supported, symmetric keys wrapped with either a symmetric or public key.

   * in some cases both authorization information and session keys needs to be transported securely from an authorization server to a resource server. In this case it would be preferable to send both in one message to avoid double headers and MACs.


Detached content

   * For the payload protected messages, the message format shall include the payload. For CoAP message protection, data which is integrity protected but not encrypted shall be detached from the message sent to avoid duplication of data already present in the message. For JWS this is known as "detached content". For JWE this is not defined but is analogous to removing the AAD from the message. Thus detached content should be allowed, but optional, since it is mainly foreseen to be applicable to CoAP message protection.


[1]: https://tools.ietf.org/html/draft-selander-ace-object-security

# snapchat-ssl-unpinning

frida scripting to beat the Snapchat SSL pinning schema. Load with frida.

## Analysis

Snapchat has been a high value target in the world of reverse engineering since the overhaul of their client software. Conventional scripting available in the frida ecosystem is not enough to circumvent protective measures built into the Snapchat client, which safeguard against common attacks performed on a victim Android system.

Disassembling the Snapchat application package for Android results in a high volume of obfuscated DEX instructions. Replicating a Snapchat API client can be considered to be a non-trivial task, and it is advisable to use machine-assisted techniques and doing analysis procedurally. For researchers working through this task, it is helpful to start by removing the CA pinning layer using a script similar to the one in this project, which will likely be patched.

The reason that Snapchat CA pinning does not work with the usual aresenal of frida scripts is that results of the internal TrustManager calls from the Java standard security implementation are hashed and hardcoded. It is possible to find the locations where the digests are stored, but it is quicker to simply use frida to overload the TrustManagerImpl#checkServerTrusted methods to capture the expected return values, then extract them into a byte representation on the host system. Once these values are retrieved from the client from normal Snapchat API usage, it is possible to pack them into a frida script similar to the one in this repository, and overload the same checkServerTrusted call to inject those return values to the host. This will allow you to use the CA from man-in-the-middle software such as mitmproxy or burp.

The Snapchat API schema was overhauled following a data breach in the early 2010s, and as of this writing (Apr 2023) most of the sensitive functionality has been replaced with a gRPC API codenamed MUSHROOM.

The source protobuf files necessary to build a custom client are possible to reverse from the DEX bytecode in the application, but these themselves are produced with an alternative build system targeting protobuf, which strips away identifying information about types and instead transpiles data structure encoders/decoders into invocations of the MessageNano class, a primitive within the protobuf internal API. Reversing the types of requests/responses would, at the very least, require tooling to interpret an AST of these non-standard encoders.

In addition, many inputs to MUSHROOM used for validation are computed within the Snapchat client and the algorithms to produce these inputs must be ascertained to fully reverse the API.

For researchers working with the Snapchat client, we invite you to forward the results of your investigations to us such that they can be shared with the intelligence community. Godspeed.

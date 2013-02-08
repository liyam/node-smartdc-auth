# Joyent SmartDC Authentication Library

Utility functions to sign http requests to SmartDC services.

    var fs = require('fs');
    var auth = require('smartdc-auth');

    var signer = auth.privateKeySigner({
          key: fs.readFileSync(process.env.HOME + '/.ssh/id_rsa', 'utf8'),
          keyId: process.env.SDC_CLI_KEY_ID,
          user: process.env.SDC_CLI_ACCOUNT
    });

## Overview

Authentication to SmartDC is built on top of Joyent's
[http-signature](https://github.com/joyent/node-http-signature) specification.
In most situations, you will only need to sign the value of the HTTP `Date`
header using your SSH private key; doing this allows you to create shell
functions to interact with SmartDC (see below).  All requests to SmartDC require an
HTTP Authorization header where the scheme is `Signature`.  Full details are
available in the `http-signature` specification, but a simple form is:

    Authorization: Signature keyId="/:login/keys/:fingerprint",algorithm="rsa-sha256" $base64_signature

The `keyId` for SmartDC is always `/$your_joyent_login/keys/$ssh_fingerprint`,
and the supported algorithms are: `rsa-sha1`, `rsa-sha256` and `dsa-sha1`.  You
then just append the base64 encoded signature.

Please, note that at the moment of writing this document, `dsa-sha1` algorithm
does not work with `sshAgentSigner` yet.

## Authenticating Requests

When creating a smartdc client, you'll need to pass in a callback function for
the `sign` parameter.  smartdc-auth ships with three functions that will likely
suit your need: `cliSigner`, `privateKeySigner` and `sshAgentSigner`.  All of
these callbacks will automatically do the correct crypto for authenticating
requests, the difference is that `privateKeySigner` expects (non-passphrase
protected) keys to be passed in directly (as a file name), whereas `cliSigner`
and `sshAgentSigner` will load your credentials on each request from the SSH
agent (if available). Both callbacks require you to set the account (login)
and keyId (SSH key fingerprint).

Note that the `cliSigner` and `sshAgentSigner` are not suitable for server
applications, or any other system where the performance degradation necessary
to interact with SSH is not acceptable; put another way, you should only use
it for interactive tooling, such as the CLI that ships with node-smartdc.

Should you wish to write a custom plugin, the expected implementation of the
`sign` callback is a function of the form `function (string, callback)`.
`string` is generated by node-smartdc (typically the value of the Date header),
and callback is of the form `function (err, object)`, where `object` has the
following properties:

    {
	      algorithm: 'rsa-sha256',   // the signing algorithm used
        keyId: '7b:c0:5c:d6:9e:11:0c:76:04:4b:03:c9:11:f2:72:7f', // key fingerprint
        signature: $base64_encoded_signature,  // the actual signature
        user: 'mark'   // the user to issue the call as.
    }

Use-cases where you would need to write your own signer are things like signing
with a smart-card or other HSM, making remote calls to a central system, etc.

## License

MIT.

## Bugs

See <https://github.com/joyent/node-smartdc-auth/issues>.

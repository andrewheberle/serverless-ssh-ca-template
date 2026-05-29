# Serverless SSH CA Template

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https%3A%2F%2Fgithub.com%2Fandrewheberle%2Fserverless-ssh-ca-template)

This is a template repository meant to be deployed to Cloudflare Workers
using the "Deploy to Cloudflare" button above.

## Deployment

Using the "Deploy to Cloudflare" button you will be requested to provide
various variables to customise the CA as follows:

```jsonc
"vars": {
    // This is the issuer of your SSH certificates
    "ISSUER_DN": "CN=SSH CA,O=Internet Widgets Pty Ltd,C=US",
    // This is the URL for the CA to verify the JWT provided by the client
    "JWT_JWKS_URL": "https://example.com/.well-known/jwks.json",
    // The issuer of the JWT access token
    "JWT_ISSUER": "https://example.com/",
    // The supported JWT algorithms as a comma seperated list
    "JWT_ALGORITHMS": "RS256",
    // An OIDC claim included in the users identity token that will be used to
    // populate the list of principals on the issued certificate
    "JWT_SSH_CERTIFICATE_PRINCIPALS_CLAIM": "groups",
    // The lifetime of the issued SSH certificates
    "SSH_CERTIFICATE_LIFETIME": "24 hours",
    // A comma seperated list of additional principals to add to all issued user certificates
    "SSH_CERTIFICATE_PRINCIPALS": "",
    // Whether to add the users own name as a valid principal
    "SSH_CERTIFICATE_INCLUDE_SELF": "false",
    // The list of SSH extensions to add to the certificate as a comma seperated list
    "SSH_CERTIFICATE_EXTENSIONS": "permit-X11-forwarding,permit-agent-forwarding,permit-port-forwarding,permit-pty,permit-user-rc",
    // A comma seperated list of users who are permitted to request SSH host certificates based on the email claim from the OIDC IdP
    "SSH_HOST_CERTIFICATE_ALLOWED_EMAILS": "",
    // The lifetime of issued Host SSH certificates in human readable form (ie "45 days"), although the client may request a shorter duration
    "SSH_HOST_CERTIFICATE_LIFETIME": "30 days",
    // The maximum time skew allowed for certificate requests
    "CERTIFICATE_REQUEST_TIME_SKEW_MAX": "90 seconds",
	// Set this to "debug" to enable more logging
	"LOG_LEVEL": "info"
},
```

The CA also requires a "Secret Store" secret to securely store the root CA
certificate:

```jsonc
"secrets_store_secrets": [
    {
        "binding": "PRIVATE_KEY",
        // The ID of the secret store
        "store_id": "<secret store id>",
        // The name of the secret
        "secret_name": "<secret name>"
    }
]
```

The secret should be an OpenSSH private key generated as follows:

```sh
ssh-keygen -t ecdsa -b 256 -f path/to/ca_key
```

At this time only ECDSA and ED25519 key types are supported for the CA,
however RSA, ECDSA and ED25519 keys are supported for users and hosts.

Finally a D1 database is require to store issued and revoked certificates:

```jsonc
"d1_databases": [
	{
		"binding": "DB",
		"database_name": "ssh-ca-database",
		"database_id": "00000000-0000-0000-0000-000000000000"
	}
]
```

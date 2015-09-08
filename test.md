RichieSSO authenticated sign-in URL for Maggio-HTML5 integrators
================================================================

The RichieSSO sign-in URL provides a secure way to provide subscriber access to
Maggio-HTML5 magazines. Your sign-in system generates a signed URL and redirects
the user to the Maggio-HTML5 authentication server, which validates the signature
and grants access to the specified magazines and magazine archives.

The sign-in URL is valid for a short time (by default 10 minutes) to prevent
distribution to third parties. It may optionally include an authenticated user
identification token to identify users engaging in prohibited behavior from
server logs.


## Recommended usage flow

We recommend the following flow on your own site to prevent easy copying
of sign-in URLs and to keep them fresh.

  1. User signs in to your site.
  
  2. Your site presents one or more available issues, with links to your
  own sign-in URL generator (for example `http://magazine.com/read/2015/5/11`).
  
  3. The sign-in URL generator checks the user's session,
  generates a sign-in URL for the current time, and immediately redirects
  the user to the generated URL. This way the sign-in URL will always be fresh
  and will not be available for casual copying.
  
  4. The Maggio-HTML5 server validates the sign-in and transfers the user to
  the specified magazine.   

## URL format
 
### Magazine sign-ins

The basic format of the sign-in URL for a magazine is `http://<hostname>/_signin/<uuid>/<timestamp>/<sig>`.
`hostname` is the Maggio-HTML5 server name for your magazine. `uuid` is UUID of the
magazine issue, which is available in the magazine index (available at `http://<hostname>/_data/index.json`). The UUID should be specified in lowercase characters. `timestamp` is the current Unix
timestamp (seconds since Jan 1 1970), without a decimal part. `sig` is the lowercase hexadecimal
representation of a HMAC signature of the URL and its parameters. The computation of the
signature is described in the section [Signature calculation](#sigcalc).

Additionally, the URL may contain both authenticated (included in the hash) and unauthenticated
(not included in the signature) query parameters. All query parameters are optional. All parameters
must be encoded as UTF-8. Conversion to Unicode Normalization Form C (the default for most systems)
is recommended where applicable (Normalization Forms are only applicable to certain non-ASCII
characters).

#### Authenticated query parameters

  * `user`: contains a user identifier. The contents not processed by the Maggio-HTML5, but
    it is included in logs for monitoring purposes.
    
  * `allow`: additional products to allow access to, if issue archives are enabled in your
    Maggio-HTML5 deployment. The value is a product key from `index.json`, such as `sample.magg.io/sample`.
    Multiple `allow` parameters may be specified. The user is implicitly granted access to
    the archive of the magazine that is being requested.

#### Unauthenticated query parameters

None.

### Archive sign-ins

To to directly enter the issue archive, use the format `http://<hostname>/_signin/archive/<timestamp>/<sig>`.
The fields are the same as for magazine sign-ins, but using one or more `allow` query parameters
is necessary if archive authentication if enabled for your products.

#### Authenticated query parameters

  * `user`: as for magazine sign-ins.
    
  * `allow`: additional products to allow access to. The value for each is a product key
    from the magazine index, such as `sample.magg.io/sample`. Multiple `allow` parameters
    may be specified. Previous access rights are revoked, so each archive sign-in should
    contain an `allow` parameter for each product the user is entitled to. 

#### Unauthenticated query parameters

  * `initial_tag`: the name of the product to show initially in the archive view. The value
    is a product key from the magazine index, such as `sample.magg.io/sample`. 

### <a name="sigcalc"></a>Signature calculation

The signature is an RFC 2104 HMAC using the SHA-256 hash function. Most standard
cryptographic libraries provide functions for computing these functions.

The key for the HMAC signature has been provided to you during Maggio-HTML5 setup.
It is typically given in the form of an UUID, such as `4361583c-be39-4dee-aa1c-a4ebe7f5ceda`.
The ASCII version of this string is the key. 

The string over which the HMAC is calculated is constructed as follows:

    issue_uuid|"archive" + "\n" +
    timestamp + "\n" +
    parameters

`issue_uuid` is the UUID of the magazine the user is entering as an ASCII string,
such as `df12727c-bd54-42be-916c-0f5dd9e8747a`, or the string `archive` when signing
an archive sign-in URL. These strings must be in lowercase characters.

`timestamp` is be the current Unix timestamp without a decimal part
converted to an ASCII string, such as `1432301730`, as it appears in the sign-in URL.

`parameters` are the *authenticated* query parameters of the URL, in UTF-8 encoded
query string format. URL encoding must *not* be applied. The parameters must be
sorted first by key and then by value. The sorting must be done after UTF-8 encoding,
in other words by sorting based on the UTF-8-encoded byte-strings, not on the original
Unicode characters. The Unicode normalization form must match that of the actual query
string (Normalization Form C is recommended).

For example, if the URL parameters are `user=foo&allow=m2&allow=m1`, the sorted parameter
string would be `allow=m1&allow=m2&user=foo`. 

The resulting signature must be converted to hexadecimal form using lowercase characters.

#### Signature calculation example

Given the above issue_uuid, timestamp and parameters, the HMAC-SHA256 would be calculated
over the following string (no trailing newline):

    df12727c-bd54-42be-916c-0f5dd9e8747a
    1432301730
    allow=m1&allow=m2&user=foo 

Given the above secret, the hex-encoded result of the calculation is `7b1ddae2592382f3cb74f15fc58df850136bfb2e180b54881545387dc2dfa10b`.

The final URL is `http://richie.example.com/_signin/df12727c-bd54-42be-916c-0f5dd9e8747a/1432301730/7b1ddae2592382f3cb74f15fc58df850136bfb2e180b54881545387dc2dfa10b?user=foo&allow=m1&allow=m2`.

#### Signature calculation example code

The following Python 3 code demonstrates computation of the signature. `auth_params` is
expected to be a list of the form `[('user', 'foo'), ('allow','m1')]`.

    def gen_hash(secret, issue_id, timestamp, auth_params=None):
        if secret is None:
            raise ValueError('secret must not be None')
        
        if isinstance(timestamp, (int, float)):
            b_timestamp = bytes(str(int(timestamp)), 'ascii')
        elif isinstance(timestamp, str):
            b_timestamp = bytes(timestamp, 'ascii')
        else:
            b_timestamp = timestamp
            
        if not isinstance(issue_id, bytes):
            b_issue_id = bytes(str(issue_id), 'ascii')
        else:
            b_issue_id = b_issue_id
            
        if not isinstance(secret, bytes):
            b_secret = bytes(str(secret), 'ascii')
        else:
            b_secret = secret
                    
        b_params = b''
        if auth_params:
            ap_list = []
            for key, value in auth_params:                
                if not isinstance(key, bytes):
                    key = bytes(key, 'utf-8')
                if not isinstance(value, bytes):
                    value = bytes(value, 'utf-8')
                
                ap_list.append((key, value))
            ap_list.sort()
            
            ap_strs = [key + b'=' + value for key, value in ap_list]                
            b_params = b'&'.join(ap_strs)
            
        sig_data = b_issue_id + b'\n' + b_timestamp + b'\n' + b_params
        
        import hashlib, hmac
        sig_hash = hmac.new(b_secret, sig_data, hashlib.sha256).hexdigest()
        
        return sig_hash
             
### Examples

The following examples demonstrate various sign-in URLs using the secret
`4361583c-be39-4dee-aa1c-a4ebe7f5ceda` signed at the time `1432301730`.

  * Sign-in to issue `de27f9d8-b020-43d7-99a6-15184d5d986f`, no parameters:
    `http://richie.example.com/_signin/de27f9d8-b020-43d7-99a6-15184d5d986f/1432301730/584345aa710a7b5ef512aa1224872f127d81950a4fff896568019cde64d5fd18`
    (signed string is `'de27f9d8-b020-43d7-99a6-15184d5d986f\n1432301730\n`).
    
  * Sign-in to issue `b46a037f-5e08-4edc-828f-35201caddd49`, one parameter:
    `http://richie.example.com/_signin/b46a037f-5e08-4edc-828f-35201caddd49/1432301730/927c8ba1b336ed4788a1a15637c8e481439d104c78a00230ce1d1c7ad13e0aac?user=foobar` (signed string is `b46a037f-5e08-4edc-828f-35201caddd49\n1432301730\nuser=foobar`).
    
  * Sign-in to issue `1e6f3357-80cc-4f54-81dc-152cc300164e`, multiple parameters:
   `http://richie.example.com/_signin/1e6f3357-80cc-4f54-81dc-152cc300164e/1432301730/fb9ed2e7e61c8abd5a680955d54f89753d9e7f1a3319694db9629e50e005306b?user=foobar&allow=m1&allow=m2`
   (signed string is `1e6f3357-80cc-4f54-81dc-152cc300164e\n1432301730\nallow=m1&allow=m2&user=foobar`).
   
  * Archive sign-in, multiple parameters, including an unauthenticated parameter:
    `http://richie.example.com/_signin/archive/1432301730/a7123bc42c5cf8be3dbaf73280e02ebb033af4d2591ebdac89d397321ee72fd4?user=foobar&allow=m1&allow=m2&initial_tag=sample.magg.io/sample`
    (signed string is `archive\n1432301730\nallow=m1&allow=m2&user=foobar`).
    
 

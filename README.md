# nginx-http-auth-totp

Time-based one-time password (TOTP) authentication for Nginx

The Time-based One-Time Password (TOTP) algorithm, provides a secure mechanism for short-lived one-time password values, which are desirable for enhanced security. This algorithm can be used across a wide range of network applications ranging from remote Virtual Private Network (VPN) access, Wi-Fi network logon to transaction-orientated Web applications.

The nginx-http-auth-totp module provides TOTP authentication for a Nginx server.

## Features

* HTTP basic authentication using time-based one-time password (TOTP)
* Cookie-based tracking of authenticated clients beyond TOTP validity window
* Configurable secret, time reference, time step and truncation length for TOTP generation
* Configurable time-skew for TOTP validation

## Build

To build the nginx-http-auth-totp module from the Nginx source directory:

    ./configure --add-module=/path/to/nginx-http-auth-totp
    make
    make install

## Configuration

    server {
        listen 80;

        location /protected {
            auth_totp_realm "Protected";
            auth_totp_file /etc/nginx/totp.conf;
            auth_totp_length 8;
            auth_totp_skew 1;
            auth_totp_step 1m;
            auth_totp_cookie "totp-session";
            auth_totp_expiry 1d;
        }
    }

## Directives

### auth_totp_cookie

* **syntax:** `auth_totp_cookie <name>`
* **default:** `totp`
* **context:** `http`, `server`, `location`, `limit_except`

Specifies the name of the HTTP cookie to be used for tracking authenticated clients.

As the validity of the time-based one-time password used for authentication expires (by design), a HTTP cookie is set following successful authentication in order to persist client authentication beyond the TOTP validity window. This configuration directives specifies the name to be used when setting this cookie while the expiry period for this cookie may be set using the `auth_totp_expiry` directive. 

### auth_totp_expiry

* **syntax:** `auth_totp_expiry <interval>`
* **default:** `0s`
* **context:** `http`, `server`, `location`, `limit_except`

Specifies the expiry time for the HTTP cookie to be used for tracking authenticated clients.

If this expiry value is not specified, the HTTP cookie used for tracking authenticated clients will be set as a session cookie which will be deleted when the current HTTP client session ends. It is important to note that the browser defines when the "current session" ends, and some browsers use session restoration when restarting, which can cause session cookies to last indefinitely.

### auth_totp_file

* **syntax:** `auth_totp_file <filename>`
* **default:** -
* **context:** `http`, `server`, `location`, `limit_except`

Specifies the file that contains usernames, shared secrets, and optionally, the UNIX start time, time step in seconds, and password truncation length, for time-based one-time password authentication. 

This configuration file has the format:

    # comment
    user1:secret1
    user2:secret2:start2
    user3:secret3:start3:step3
    user4:secret4:start4:step4:length4
    user5:secret5

If the UNIX start time (in seconds since 1970/01/01), the time step size (in seconds), or the password truncation length, is specified in association with a user definition within this file, these values will override any values defined independently via the configuration directives `auth_totp_start`, `auth_totp_step` or `auth_totp_length`.

If the UNIX start time is specified - either within the TOTP definition file or by way of the `auth_totp_start` configuration directive - and represents a time in the future, the authentication request for the given user account will fail.

### auth_totp_length

* **syntax:** `auth_totp_length <number>`
* **default:** `6`
* **context:** `http`, `server`, `location`, `limit_except`

Specifies the truncation length of the Time-based One-Time Password (TOTP) code. This truncation length may be between 1 and 8 digits inclusively.

If the supplied TOTP is of a different length to this value, the authentication request will fail.

### auth_totp_realm

* **syntax:** `auth_totp_realm <string>|off`
* **default:** `off`
* **context:** `http`, `server`, `location`, `limit_except`

Enables validation of user name and time-based one-time password using the "HTTP Basic Authentication" protocol. The specified parameter is used as the `realm` for this authentication. This parameter value can contain variables. The special value of `off` cancels the application of any `auth_totp_realm` directive inherited from a higher configuration level.

### auth_totp_skew

* **syntax:** `auth_totp_skew <number>`
* **default:** `1`
* **context:** `http`, `server`, `location`, `limit_except`

Specifies the number of time steps by which the time base between the issuing and validating TOTP systems.

Due to network latency, the gap between the time that a OTP was generated and the time that the OTP is received at the validating system may be large. Indeed, it is possible that the receiving time at the validating system and that when the OTP was generated by the issuing system may not fall within the same time-step window. Accordingly, the validating system should typically set a policy for an acceptable OTP transmission window for validation. In line with this, the validating system should compare OTPs not only with the receiving timestamp, but also the past timestamps that are within the transmission delay.

It is important to note that larger acceptable delay windows represent a larger window for attacks and a balance must be struck between the security and usability of OTPs.

### auth_totp_start

* **syntax:** `auth_totp_start <time>`
* **default:** `0`
* **context:** `http`, `server`, `location`, `limit_except`

Specifies the UNIX time from which to start counting time steps as part of TOTP algorithm operations.

The default value is 0, the UNIX epoch at 1970/01/01. 

If the UNIX start time specified - either within this directive or in the configuration file specified by the `auth_totp_file` directive - represents a time in the future, authentication will be denied.

### auth_totp_step

* **syntax:** `auth_totp_step <interval>`
* **default:** `30s`
* **context:** `http`, `server`, `location`, `limit_except`

Specifies the time step as part of TOTP algorithm operations.

## References

* [RFC 4226 HOTP: An HMAC-Based One-Time Password Algorithm](https://datatracker.ietf.org/doc/html/rfc4226)
* [RFC 6238 TOTP: Time-Based One-Time Password Algorithm](https://datatracker.ietf.org/doc/html/rfc6238)
* [RFC 7235 Hypertext Transfer Protocol (HTTP/1.1): Authentication](https://datatracker.ietf.org/doc/html/rfc7235)


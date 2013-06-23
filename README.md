This "spec" enables users to use their preferred email address when logging into a Persona-enabled site and authenticate using IndieAuth.

# User setup

Here are the things that users must do to delegate Persona:

* serve a support document at `https://example.com/.well-known/browserid` that sets `indieauth.com` as the authority
* include a `rel="me"` email address on their personal domain
* setup WebFinger on their email domain to point to their personal domain (if email and personal domains are not the same, eg. `aaron@parecki.com` v. `aaronparecki.com`)

# Walkthrough

1. User types in `aaron@parecki.com`.
2. Persona looks up support document at `https://parecki.com/.well-known/browserid`.
3. The support document contains `"authority": "indieauth.com"`.
4. Persona calls IndieAuth's provisioning page asking it to sign `aaron@parecki.com`'s cert which will fail because that user doesn't have a session with IndieAuth.
5. Persona calls IndieAuth's login page with `aaron@parecki.com` as a param.
6. Persona calls IndieAuth's provisioning page asking it to sign `aaron@parecki.com`'s cert which will succeeed.

## IndieAuth "login"

1. IndieAuth receives a request to log `aaron@parecki.com` in.
2. It needs to translate that to a domain name and present the regular IndieAuth login page (without the option to use an email for login?).
3. Once it has the domain, IndieAuth sets a cookie for `aaron@parecki.com` valid for 5 minutes.

### rel=me lookup

1. IndieAuth extracts the domain part of the email address.
2. It looks for a `rel="me"` link containing `aaron@parecki.com` on `https://parecki.com`.
3. If it's there, it uses that for the auth, otherwise IndieAuth does a webfinger lookup.

### WebFinger lookup (old-style)

1. IndieAuth looks at `https://parecki.com/.well-known/host-meta`.
2. Extracts the lrdd temlate and calls it.
3. Parses the XRD response and finds the `<link rel="me">` tag.
4. If the tag is missing or there's more than one, abort.
5. Uses the domain found in step 3 for the auth.

### WebFinger lookup (new-style)

1. IndieAuth lookups email at `https://parecki.com/.well-known/webfinger?resource=acct:aaron@parecki.com`.
2. Extracts the `"rel": "me"` link out of the list of links.
3. If that link is missing or there's more than one, abort.
4. Uses the domain found in step 2 for the auth.

### IndieAuth session cookie

The session that IndieAuth creates with the client can be entirely client-side (signed with a private key owned by IndieAuth) and it contains:

* the email address of the user
* date after which the cookie should no longer be honoured (5 minutes later)
* separate expiry header to let the browser clean up old unusable cookies

## IndieAuth provisioning

1. IndieAuth receives a request to sign a cert for `aaron@parecki.com`.
2. IndieAuth notices that there's an unexpired cookie signed with its private key.
3. If the email in the signed cookie matches the one requested by Persona, a signed Persona cert is returned with an undefined (but longer than 5 minutes) validity.

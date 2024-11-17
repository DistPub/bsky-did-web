# didweb

> ⚠️WIP

`didweb` is a simple utility that helps to migrate you DID to [Bluesky PDS](https://github.com/bluesky-social/pds).

## Compiling
```
go build -o didweb main.go
```

## Example
```bash
PRIVKEY=$(didweb genkey)
PUBKEY=$(echo -n $PRIVKEY | didweb pubkey)
# the keys only used to create account, after create account, the PDS will hold another keys

didweb gendid --handle alice.domain.tld --pubkey $PUBKEY --hostname pds.example.com > did.json
# upload this did.json file to your server, for example, https://alice.domain.tld/.well-known/did.json

# createInviteCode
curl   --fail   --silent   --show-error   --request POST   --user "admin:REPLACE_WITH_YOUR_PDS_ADMIN_PASSWORD"   --header "Content-Type: application/json"   --data '{"useCount": 1}'   "https://pds.example.com/xrpc/com.atproto.server.createInviteCode"
# if you want use multiple times to create multiple accounts, increase the useCount field value

# setup handle verify
# NOTE: if your handle is not ends with hostname(handle != *.pds.example.com) make sure reference to real did:web account before call create account
# for example: alice.me
# 
# Upload alice.me/.well-known/atproto-did file with content: did:web:alice.domain.tld
# or Add DNS TXT record ("_atproto.alice.me") with content: did=did:web:alice.domain.tld
# so that the client can validate your did:web account.

# now you can try to sign up
didweb sign --privkey $PRIVKEY --iss did:web:alice.domain.tld --aud did:web:pds.example.com --exp 180\
    | didweb createAccount --pds https://pds.example.com --did did:web:alice.domain.tld --handle alice.domain.tld --invite the-invite-code --email youremail@outlook.com --password setyourpassword

# Or, you can directly call createAccount API if the above `didweb createAccount` does not work
TOKEN=$(./didweb sign --privkey $PRIVKEY --iss did:web:alice.domain.tld --aud did:web:pds.example.com --exp 180)
curl --verbose  --fail   --silent   --show-error   --request POST --header "Authorization: Bearer $TOKEN"  --header "Content-Type: application/json"   --data "{\"email\":\"youremail@outlook.com\", \"handle\":\"alice.domain.tld\", \"did\":\"did:web:alice.domain.tld\", \"password\":\"setyourpassword\", \"inviteCode\":\"the-invite-code\"}"   "https://pds.example.com/xrpc/com.atproto.server.createAccount"

# the PDS will verify token by use public key from https://alice.domain.tld/.well-known/did.json, if pass, PDS will check invite code, if pass, new repo related to did:web:alice.domain.tld will create, also create private key and public key


# now you will get new JWT token to complete registration
TOKEN=accessJwt_value_from_response

# getRecommendedDidCredentials
curl --verbose  --fail   --silent   --show-error   --request GET --header "Authorization: Bearer $TOKEN" https://pds.example.com/xrpc/com.atproto.identity.getRecommendedDidCredentials

# -> edit your did.json:
# Replace the value of "publicKeyMultibase" in your did.json with the value of 
# verificationMethods.atproto in the response of getRecommendedDidCredentials. 
# Don't forget to remove "did:key" from the value when replacing.
# If you changed handle to different domian, make sure also update the alsoKnownAs field

# activateAccount
curl --verbose  --fail   --silent   --show-error   --request POST --header "Authorization: Bearer $TOKEN" https://pds.example.com/xrpc/com.atproto.server.activateAccount

# now, your account has been activated.
```

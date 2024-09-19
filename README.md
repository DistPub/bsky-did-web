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
didweb gendid --handle alice.domain.tld --pubkey $PUBKEY --hostname pds.example.com > did.json
# upload this did to your .well-known directory

# createInviteCode
curl   --fail   --silent   --show-error   --request POST   --user "admin:REPLACE_WITH_YOUR_PDS_ADMIN_PASSWORD"   --header "Content-Type: application/json"   --data '{"useCount": 1}'   "https://pds.example.com/xrpc/com.atproto.server.createInviteCode"

# now you can try to sign up
didweb sign --privkey $PRIVKEY --iss did:web:alice.domain.tld --aud did:web:pds.domain.tld --exp 180 | didweb createAccount --pds https://pds.domain.tld --handle alice.domain.tld --invite pds-domain-tld-invite-code --email alice@domain.tld --password password123

# Or, you can directly call createAccount API if the above `didweb createAccount` does not work
TOKEN=$(./didweb sign --privkey $PRIVKEY --iss did:web:did.example.com --aud did:web:pds.example.com --exp 180)
curl --verbose  --fail   --silent   --show-error   --request POST --header "Authorization: Bearer $TOKEN"  --header "Content-Type: application/json"   --data "{\"email\":\"youremail@outlook.com\", \"handle\":\"alice.domain.tld\", \"did\":\"did:web:alice.domain.tld\", \"password\":\"XXX_REPLACE_THIS_XXX\", \"inviteCode\":\"pds-invite-replace-this\"}"   "https://pds.example.com/xrpc/com.atproto.server.createAccount"


# now you will get new JWT token to complete registration
TOKEN=accessJwt_value_from_response

# getRecommendedDidCredentials
curl --verbose  --fail   --silent   --show-error   --request GET --header "Authorization: Bearer $TOKEN" https://pds.example.com/xrpc/com.atproto.identity.getRecommendedDidCredentials

# -> edit your did.json:
# Replace the value of "publicKeyMultibase" in your did.json with the value of 
# verificationMethods.atproto in the response of getRecommendedDidCredentials. 
# Don't forget to remove "did:key" from the value when replacing.

# activateAccount
curl --verbose  --fail   --silent   --show-error   --request POST --header "Authorization: Bearer $TOKEN" https://pds.example.com/xrpc/com.atproto.server.activateAccount
```

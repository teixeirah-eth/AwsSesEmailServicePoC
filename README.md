# AWS SES Email Service PoC


## AWS SES
We need access and secret keys with the following actions for the application in order to use SES:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "*"
        }
    ]
}
```
 Also, a verified email address or domain is necessary. It can easily be done by following instructions in the AWS Console. Ir order to verify the domain, it is required some extra steps since it is based on DomainKeys Identified Mail (DKIM). It will also be necessary to ask AWS for production access in the SES service. AWS does this in order to prevent new accounts to spam people.

## Email verification

In order to verify the user's email, we can generate a code that is sent by emial to the user. Here's a diagram that shows the sequence of actions:
```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Registry
    participant EmailService
    participant Database
    
    User->>Frontend: Click verify email
    Frontend->>Registry: Request to verify email 
    Registry->>Registry: Generates a code associated with that email
    Registry->>Database: Persist the code linked with that email
    Registry->>EmailService: Request to send email with code
    EmailService->>User: Sends email with code
    User->>Frontend: Input code
    Frontend->>Registry: Request to validate code
    Registry->>Database: Retrieves code by email and validates
    Registry->>Frontend: Return success or failure
    

```


## Apache FreeMarker

In order to generate the email, I've used Apache FreeMarker library for HTML parsing. Variables can be replaced by using this syntax:
```html
<p>Hello ${name}! This is the requested code: ${code}</p>
```
And in java:
```java
Map<String, Object> model = new HashMap<>();
      model.put("name", "Gustavo Teixeira");
      model.put("code", "548621");
      String html = FreeMarkerTemplateUtils.processTemplateIntoString(t, model);
```


## Results

With this version of the code, this is the email that was sent to my inbox:
![image](https://user-images.githubusercontent.com/10344032/227000726-c788a7af-2b58-4c30-92d0-12951529ff12.png)

I've used the following request to trigger the email:
```bash
curl -i -X POST -H "Content-Type:application/json"  -d  '{ "fromEmail": "contact@gustavoteixeira.dev", "toEmail": "iwhrim@gmail.com", "subject": "test email", "body": "Hi, This is a test email." }'  'http://localhost:8080/sendEmail'
```

# Testing GH Copilot to catch erroneous config updates

A basic readme was generated and put in `.github/copilot-instructions.md` to guide Copilot in deciding when something could potentially be bad for configs, ie:
 - Updating something that shouldnt have been
 - A value that is for a different env
 - Something that may be useful to put in UAT but wouldnt make sense for dev

## Response time

Illustraded is a response generated less than a minute after the PR was created (this is running on my personal account, so public runners were used).
 ![Effective response time](./_progress/response_time.png)
[Link to PR](https://github.com/christoffellis/test-copilot-repo/pull/3)

## Adding a new env

In another test, a new env was added, with only half the values were updated. 

[Link to PR](https://github.com/christoffellis/test-copilot-repo/pull/4)

## Testing in ONLY UAT

For this PR, we're testing a config flag in UAT. However, we're simulating that we needed some configs for dev, but they shouldnt be pushed to the actual repo. Further, a stray SIT variable was renamed to mimic typical accidents we can sometimes push.

```
Comments suppressed due to low confidence (1)
sit/config.yaml:67

Adding 'https://localhost:3000' to CORS allowed origins in the SIT environment is inappropriate. Localhost should not be using HTTPS in typical development scenarios (localhost typically uses HTTP), and this creates an inconsistent security configuration. If localhost access is needed for testing, this should be 'http://localhost:3000' or removed entirely as localhost access is typically only appropriate for dev environments.
```
Here it caught that localhost was innapropriate for SIT, however did not catch the fact that this PR was only for UAT. We can maybe add more instructions to avoid this. 

```
The database connection string still references the old password 'uat_password' but the password field was updated to '8sIas&s]12*st'. This creates a mismatch where the application will fail to connect to the database. Update the connection string to use the new password value.
```

It also spotted a variable that should be the same as another, but wasnt updated.
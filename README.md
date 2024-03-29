# A guide to modifing Microsoft Teams

> IMPORTANT: This is based on the version of Teams that uses Electron. This is NOT for the new (Windows 11) version of Teams.

**Note: The Wiki contains the original Guide. The guide in the Wiki documents modifiyng Teams via JS using the Developer tools. However, I have wanted to experiment with the (web) API that Teams uses for server-based stuff (e.g. setting your status). The web API stuff is in the readme file. I am (mostly) focusing on the web API, but feel free to ask me questions about either one in the Issues tab.**

First, a little background on what this is. It is a guide to what I have figured out in Microsoft Teams. I wanted to do (a) control the Teams app with JS and (b) use the Teams web API. I was able to open the developer tools, but I was unable to find any useful reference on the internal HTML/API or how it works. In the Wiki, I attempt to document everything I have figured out how to do _only using the JavaScript console in the DevTools_. This is _not_ a reference guide or documentation, but it does contain what I have figured out how to do. This repo is meant as a cookbook, of (hopefully) useful things you can do. All of these experiments assume you are in the JavaScript console within DevTools. So read Opening DevTools before you try these. The stuff on the web API is here in the readme.

## Hey! Where are the tutorials?

For JS, in the Wiki. For the web API, here in the readme.

## Web API Information

For each API example, I explain the web request paramaters, and an example of using it in PowerShell using `Invoke-WebRequest`

### Setting your Teams status

This one is pretty simple. Note that when you make the web request, you might have to close/reopen Teams to see teh change. This is because Teams isn't constantly checking for a status change. However, the status is still set _even if you don't see it in Teams_.

Make a `PUT` request to `https://presence.teams.microsoft.com/v1/me/forceavailability/`. The authorization should be `BEARER <YOUR AUTH TOKEN HERE>`. The content type should be `application/json`. Have the body of the request have `availability = "<STATUS>"`. `STATUS` can be `Busy` or `Available`. I haven't figured out the `STATUS` for Do Not Disturb, Be Right Back, Appear away, Appear offline.

A PowerShell sample is
```PowerShell
$data = @{
    availability = "Busy"
} | ConvertTo-Json

Invoke-WebRequest https://presence.teams.microsoft.com/v1/me/forceavailability/ -Method PUT -Headers @{Authorization = "Bearer <YOUR AUTH TOKEN HERE"} -Body $data -ContentType "application/json"
```

To find your Bearer/auth token, open the Teams developer tools, go to the Network tab, and set your status to something that it isn't already (e.g. Busy). There will be a request called `forceavailability`. Click on it, and go to the Headers tab (for that request). There will be a property called `authorization`, with a token that looks like `Bearer` and a ton of random letters. This is your authorization token. A sample Authorization for the request header is `Bearer XXXX`, where `XXXX` is your token. Double check that it is `Bearer XXXX` not `Bearer Bearer XXXX`, or else your request will fail.

### Setting your Teams status message

Make a `PUT` request to `https://presence.teams.microsoft.com/v1/me/publishnote`. Use the same authorization sceme as for setting your status (`BEARER <YOUR AUTH TOKEN HERE>`). Again, to find your auth token, use the method described in the last section. The body of the request should have a `message` of whatever your status should be (e.g. `"This is a sample status message"`), and an `expiry` of `"9999-12-31T08:00:00.000Z"` to make the message permenent (until you change it to something else).

A PowerShell example is
```PowerShell
$data = @{
    message = "This is a sample status message"
    expiry = "9999-12-31T08:00:00.000Z"
} | ConvertTo-Json

Invoke-WebRequest https://presence.teams.microsoft.com/v1/me/publishnote -Method PUT -Headers @{Authorization = "Bearer <YOUR AUTH TOKEN HERE>"} -Body $data -ContentType "application/json"
```

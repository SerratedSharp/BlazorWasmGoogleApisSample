# Blazor Standalone WASM  integrated with Google APIs
This sample displays a Google Sign in button, completes OIDC authentication, requests an authorization token for a specific scope, and performs an action by writing to Google Drive AppDataFolder with that authorization.

There is little about this sample that is specific to Blazor WASM, other than to demonstrate a verified working approach to help others cut through the plethora of different libraries and deprecated documentation that can lead you down rabbit holes.  Because Blazor Standalone WASM runs client-side, then we use the client-side JavaScript library to initialize the Sign-in button and handle sign-in responses.  This could easily be expanded to include callbacks into C# code or wrapped in an interop wrapper. 

As detailed in source code, you must provide your own Google OIDC ClientID and configure to allow redirects to your localhost URL if testing locally.

## Implementation

Import the Google Identity Services API library.  This sample includes it in `/wwwroot/index.html`:
```
<script src="https://accounts.google.com/gsi/client" async defer></script>
```

> [!IMPORTANT] 
> **The rest of the implementation can be found in [Pages/Home.razor](https://github.com/SerratedSharp/BlazorWasmGoogleApisSample/blob/main/Pages/Home.razor)**

## Google Cloud Configuration

These instructions are very brief, as there's plenty of documentation with more details, but I wanted to cover the core steps to get to a working OIDC integration. 

- Sign-in to https://console.cloud.google.com/
- Create a new Project
- Go to APIs & Services, enable the API you wish to use, in this case Google Drive API
- Go to APIs & Services -> Credentials
  - ![image](https://github.com/user-attachments/assets/ea887de1-ab36-464e-bbae-35be886cdd8c)
- Click Create Credentials -> OAuth Client ID
  - ![image](https://github.com/user-attachments/assets/6f070cec-f2b1-44d5-85bf-27c90021b658)
- Choose "Web Application"
  - ![image](https://github.com/user-attachments/assets/35bcd7d5-d80b-42eb-aed3-8435df159e8d)

Fill in the details.  It's advisable to have seperate registrations or projects for different environments.  For your dev registration you will need to enter your `https://localhost:1324` with the given port your project launches with.  Enter these for both Javascript origins and redirect URIs. (Observe the note about how long it takes this configuration to take effect.  I have observed it to be very quick, but could potentially take a couple hours.)   Completing this will generate the Client ID which you pass in the appropriate places as described in the [Pages/Home.razor](https://github.com/SerratedSharp/BlazorWasmGoogleApisSample/blob/main/Pages/Home.razor) sample.  You do not need the client secret since this is a client-side implicit flow.

You will also need to configure an OAuth consent screen:

![image](https://github.com/user-attachments/assets/6610a330-23f4-4bfc-84f7-c5f3c57093dc)

- The App Domain entered here can be your production domain name.  You do not need to enter localhost here.
- On the second page you add scopes that your app might request.  For example, we added the `drive.appdata` scope for this sample:
  - ![image](https://github.com/user-attachments/assets/05e03d2c-cd39-4041-b8bb-a3fd573abbd5)
  - Scopes will not be available to select here unless the parent service it falls under is enabled under APIs & Services first. For example, we previously enabled the Google Drive API.

**Important:** Google will not allow you to sign in from a browser that has a debugger attached.  It will instead show a warning that the user may be using a compromised browser.  When you launch your project, you'll need to copy you localhost:port URL from the spawned browser instance, and paste it into either your regular browser or an incognito instance before you can complete the sign in.  

This sample immediately chains an action of writing a file after signing in.  Thus you get the Sign-In dialog, followed by the authorization consent dialog asking for the AppData folder permission.  Typically the user would click the sign-in button, and then later when you need to attempt an action, they would then get the second dialog.  This is known as incremental authorization, and ensures your application doesn't request excessive permissions for features the user may not leverage.







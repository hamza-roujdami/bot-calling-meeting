# Microsoft Teams Calling Bot Sample

A Teams bot that demonstrates calling and meeting capabilities using the Microsoft Bot Framework and Microsoft Graph API.

## Features

- **Voice Calling**: Make and receive voice calls in Microsoft Teams
- **Meeting Integration**: Join and manage Teams meetings
- **Adaptive Cards**: Interactive messaging with adaptive cards
- **Speech Recognition**: Optional speech-to-text capabilities using Azure Cognitive Services
- **Call Recording**: Record and download call recordings

## Prerequisites

- .NET 6.0 SDK
- Azure subscription
- Microsoft Teams account
- ngrok (for local development)
- Azure AD app registration
- Azure Bot Service

## Configuration

### 1. Azure AD App Registration

Create an Azure AD app with the following:
- App Type: **SingleTenant**
- Redirect URIs configured
- Client secret generated
- API Permissions:
  - Microsoft Graph:
    - `User.Read`
    - `Application.ReadWrite.All`
    - `Calls.Initiate.All`
    - `Calls.JoinGroup.All`
    - `OnlineMeetings.ReadWrite.All`

### 2. Azure Bot Service

Create an Azure Bot Service with:
- SKU: S1
- App Type: SingleTenant
- Messaging endpoint: `https://YOUR_NGROK_URL/api/messages`
- Teams channel enabled with calling support
- Calling webhook: `https://YOUR_NGROK_URL/api/calling`

### 3. Application Configuration

Update `appsettings.json` with your credentials:

```json
{
  "MicrosoftAppType": "SingleTenant",
  "MicrosoftAppId": "YOUR_APP_ID",
  "MicrosoftAppPassword": "YOUR_APP_SECRET",
  "MicrosoftAppTenantId": "YOUR_TENANT_ID",
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "YOUR_TENANT_ID",
    "ClientId": "YOUR_APP_ID",
    "ClientSecret": "YOUR_APP_SECRET"
  },
  "Bot": {
    "AppId": "YOUR_APP_ID",
    "AppSecret": "YOUR_APP_SECRET",
    "PlaceCallEndpointUrl": "https://graph.microsoft.com/v1.0",
    "BotBaseUrl": "YOUR_NGROK_URL",
    "GraphApiResourceUrl": "https://graph.microsoft.com",
    "MicrosoftLoginUrl": "https://login.microsoftonline.com/",
    "RecordingDownloadDirectory": "temp",
    "CatalogAppId": "YOUR_APP_ID"
  }
}
```

## Running Locally

1. **Start ngrok**:
   ```bash
   ngrok http 3978
   ```

2. **Update configuration**:
   - Copy the ngrok URL
   - Update `BotBaseUrl` in `appsettings.json`
   - Update Azure Bot Service messaging endpoint
   - Update `validDomains` in `AppManifest/manifest.json`

3. **Run the application**:
   ```bash
   cd csharp/Source/CallingBotSample
   dotnet run
   ```

4. **Deploy to Teams**:
   - Create the manifest.zip:
     ```bash
     cd AppManifest
     zip -r ../manifest.zip .
     ```
   - Upload `manifest.zip` to Teams (Apps → Upload a custom app)

## Project Structure

```
bot-calling-meeting/
├── csharp/
│   └── Source/
│       └── CallingBotSample/
│           ├── AdaptiveCards/        # Adaptive card templates
│           ├── AppManifest/          # Teams app manifest
│           ├── Authentication/       # Authentication providers
│           ├── Bots/                 # Bot logic (MessageBot, CallingBot)
│           ├── Cache/                # Caching services
│           ├── Controllers/          # API controllers
│           ├── Options/              # Configuration options
│           ├── Services/             # Services (Graph, Speech, etc.)
│           ├── wwwroot/              # Static files
│           ├── appsettings.json      # Application configuration
│           └── Startup.cs            # Application startup
└── README.md
```

## Key Components

### Bots
- **MessageBot**: Handles text-based messaging and adaptive cards
- **CallingBot**: Manages voice calls and call notifications

### Services
- **CallService**: Microsoft Graph calling API integration
- **ChatService**: Teams chat operations
- **OnlineMeetingService**: Meeting management
- **SpeechService**: Azure Cognitive Services speech recognition
- **TeamsRecordingService**: Call recording handling

### Authentication
- **ConfigurationBotFrameworkAuthentication**: Bot Framework auth
- **ClientSecretCredential**: Azure AD authentication with client secrets
- **AuthenticationProvider**: Custom auth provider for Graph API calls

## Troubleshooting

### 401 Unauthorized Errors
- Verify `MicrosoftAppId`, `MicrosoftAppPassword`, and `MicrosoftAppTenantId` match your Azure AD app
- Ensure `MicrosoftAppType` is set to `"SingleTenant"`
- Check that Azure Bot Service has the app password configured
- Verify API permissions are granted admin consent

### ngrok Issues
- Free ngrok accounts are limited to 1 simultaneous session
- ngrok URLs change each time you restart - update all configurations
- Add `ngrok-skip-browser-warning: true` header for API testing

### Teams Manifest Errors
- Ensure `id` and `botId` match your Azure AD app ID
- Update `validDomains` with your current ngrok domain (without https://)
- Verify all required permissions are listed

## Security Notes

**⚠️ IMPORTANT**: Never commit sensitive credentials to version control!

- Add `appsettings.Development.json` to `.gitignore`
- Use environment variables for production deployments
- Rotate client secrets regularly
- Store secrets in Azure Key Vault for production

## Technologies Used

- .NET 6.0
- ASP.NET Core
- Microsoft Bot Framework SDK
- Microsoft Graph SDK
- Azure Cognitive Services (optional)
- Azure Identity

## License

This project is based on the [Microsoft Teams Samples](https://github.com/OfficeDev/Microsoft-Teams-Samples) repository.

## Support

For issues and questions:
- Check the [Bot Framework documentation](https://docs.microsoft.com/en-us/azure/bot-service/)
- Review [Microsoft Graph calling API docs](https://docs.microsoft.com/en-us/graph/api/resources/calls-api-overview)
- Visit [Microsoft Teams developer docs](https://docs.microsoft.com/en-us/microsoftteams/platform/)


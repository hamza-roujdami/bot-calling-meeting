# Quick Setup Guide

This guide will help you get the Teams Calling Bot running on your machine.

## Prerequisites

Before you begin, ensure you have:

- [ ] .NET 6.0 SDK installed ([Download](https://dotnet.microsoft.com/download/dotnet/6.0))
- [ ] ngrok account ([Sign up](https://ngrok.com/))
- [ ] Azure subscription
- [ ] Microsoft Teams account
- [ ] Azure CLI installed (optional, but recommended)

## Step-by-Step Setup

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/hamza-roujdami/bot-calling-meeting.git
cd bot-calling-meeting/csharp/Source/CallingBotSample
```

### 2️⃣ Create Azure AD App Registration

Using Azure Portal:

1. Go to [Azure Portal](https://portal.azure.com) → Azure Active Directory → App registrations
2. Click **New registration**
3. Fill in:
   - Name: `Teams Calling Bot`
   - Supported account types: **Accounts in this organizational directory only (Single tenant)**
4. Click **Register**
5. Copy the **Application (client) ID** - you'll need this
6. Copy the **Directory (tenant) ID** - you'll need this

### 3️⃣ Create Client Secret

1. In your app registration, go to **Certificates & secrets**
2. Click **New client secret**
3. Description: `Bot Secret`
4. Expires: Choose your preferred duration
5. Click **Add**
6. **IMPORTANT**: Copy the secret VALUE immediately (it won't be shown again)

### 4️⃣ Configure API Permissions

1. In your app registration, go to **API permissions**
2. Click **Add a permission** → **Microsoft Graph** → **Application permissions**
3. Add these permissions:
   - `User.Read`
   - `Application.ReadWrite.All`
   - `Calls.Initiate.All`
   - `Calls.JoinGroupCall.All`
   - `Calls.InitiateGroupCall.All`
   - `OnlineMeetings.ReadWrite.All`
4. Click **Grant admin consent for [Your Organization]**

### 5️⃣ Create Azure Bot Service

Using Azure CLI:

```bash
# Login to Azure
az login

# Set variables
RESOURCE_GROUP="your-resource-group"
BOT_NAME="teams-calling-bot-$(date +%s)"
APP_ID="YOUR_APP_ID_FROM_STEP_2"
TENANT_ID="YOUR_TENANT_ID_FROM_STEP_2"

# Create bot service
az bot create \
  --app-type SingleTenant \
  --appid $APP_ID \
  --name $BOT_NAME \
  --resource-group $RESOURCE_GROUP \
  --sku S1 \
  --location global \
  --tenant-id $TENANT_ID

# Update endpoint (replace with your ngrok URL when you get it)
az bot update \
  --name $BOT_NAME \
  --resource-group $RESOURCE_GROUP \
  --endpoint "https://YOUR_NGROK_URL/api/messages"

# Enable Teams channel with calling
az bot msteams create \
  --name $BOT_NAME \
  --resource-group $RESOURCE_GROUP \
  --enable-calling true \
  --calling-web-hook "https://YOUR_NGROK_URL/api/calling" \
  --location global
```

### 6️⃣ Setup Local Configuration

1. **Copy the template file**:
   ```bash
   cp appsettings.template.json appsettings.json
   ```

2. **Edit `appsettings.json`** with your values:
   ```json
   {
     "MicrosoftAppType": "SingleTenant",
     "MicrosoftAppId": "YOUR_APP_ID",
     "MicrosoftAppPassword": "YOUR_CLIENT_SECRET",
     "MicrosoftAppTenantId": "YOUR_TENANT_ID",
     "AzureAd": {
       "TenantId": "YOUR_TENANT_ID",
       "ClientId": "YOUR_APP_ID",
       "ClientSecret": "YOUR_CLIENT_SECRET"
     },
     "Bot": {
       "AppId": "YOUR_APP_ID",
       "AppSecret": "YOUR_CLIENT_SECRET",
       "BotBaseUrl": "https://YOUR_NGROK_URL",
       "CatalogAppId": "YOUR_APP_ID"
     }
   }
   ```

### 7️⃣ Start ngrok

```bash
ngrok http 3978
```

Copy the HTTPS forwarding URL (e.g., `https://abc123.ngrok-free.app`)

### 8️⃣ Update Configurations with ngrok URL

1. **Update `appsettings.json`**:
   - Set `Bot.BotBaseUrl` to your ngrok URL

2. **Update Azure Bot Service** (replace with your actual values):
   ```bash
   az bot update \
     --name YOUR_BOT_NAME \
     --resource-group YOUR_RESOURCE_GROUP \
     --endpoint "https://YOUR_NGROK_URL/api/messages"
   ```

3. **Update Teams manifest**:
   - Edit `AppManifest/manifest.json`
   - Replace `YOUR_AZURE_AD_APP_ID` with your App ID (3 places: `id`, `botId`, `webApplicationInfo.id`)
   - Replace `YOUR_NGROK_DOMAIN.ngrok-free.app` in `validDomains` (without `https://`)

### 9️⃣ Run the Bot

```bash
cd csharp/Source/CallingBotSample
dotnet run
```

You should see:
```
Now listening on: http://localhost:3978
Application started. Press Ctrl+C to shut down.
```

### 🔟 Create and Deploy Teams Manifest

1. **Create the manifest.zip**:
   ```bash
   cd AppManifest
   zip -r ../manifest.zip .
   ```

2. **Upload to Teams**:
   - Open Microsoft Teams
   - Go to **Apps** → **Manage your apps**
   - Click **Upload an app** → **Upload a custom app**
   - Select the `manifest.zip` file
   - Click **Add**

3. **Test the bot**:
   - Send a message: `hello`
   - You should see a welcome card with options!

## Troubleshooting

### Bot returns 401 Unauthorized

**Problem**: Authentication failing

**Solution**:
1. Verify `MicrosoftAppType` is set to `"SingleTenant"` in `appsettings.json`
2. Ensure `MicrosoftAppTenantId` is set correctly
3. Check that all App IDs match across:
   - `appsettings.json`
   - `manifest.json` (3 places)
   - Azure Bot Service
   - Azure AD app registration

### Bot not receiving messages

**Problem**: ngrok or endpoint issues

**Solution**:
1. Verify ngrok is running: `curl http://localhost:4040/api/tunnels`
2. Check ngrok URL is updated in:
   - `appsettings.json` → `Bot.BotBaseUrl`
   - Azure Bot Service messaging endpoint
   - `manifest.json` → `validDomains`
3. Test ngrok forwarding: `curl https://YOUR_NGROK_URL/`

### AADSTS700016 Error

**Problem**: App not found in Bot Framework directory

**Solution**:
1. Ensure Azure AD app is created as **SingleTenant** (not MultiTenant)
2. Verify `MicrosoftAppTenantId` matches your tenant ID
3. Check API permissions are granted admin consent
4. Ensure Azure Bot Service has the same tenant ID configured

### Teams won't install the app

**Problem**: Manifest validation errors

**Solution**:
1. Verify all App IDs in `manifest.json` match your Azure AD app
2. Ensure `validDomains` contains your ngrok domain (without `https://`)
3. Check manifest schema version is supported
4. Validate manifest at: [Teams App Validator](https://dev.teams.microsoft.com/appvalidation.html)

## Security Best Practices

⚠️ **IMPORTANT**: 

1. **Never commit `appsettings.json`** - it's already in `.gitignore`
2. **Rotate client secrets regularly** (every 90 days recommended)
3. **Use Azure Key Vault** for production deployments
4. **Enable Azure AD Conditional Access** for enhanced security
5. **Monitor app permissions** and remove unused ones

## Testing Checklist

- [ ] Bot responds to messages in Web Chat
- [ ] Bot responds to messages in Teams
- [ ] Can create a call using adaptive cards
- [ ] Can join a Teams meeting
- [ ] Calling webhook receives notifications at `/callback`
- [ ] Audio prompts work correctly
- [ ] Call recording (if enabled) saves files to `temp/` directory

## Quick Reference

**ngrok Dashboard**: http://localhost:4040  
**Bot local URL**: http://localhost:3978  
**Azure Portal**: https://portal.azure.com  
**Teams Dev Portal**: https://dev.teams.microsoft.com  

## Need Help?

- Check the [main README](README.md) for detailed documentation
- Review [Bot Framework docs](https://docs.microsoft.com/en-us/azure/bot-service/)
- See [Microsoft Graph calling API](https://docs.microsoft.com/en-us/graph/api/resources/calls-api-overview)
- Visit [Microsoft Teams platform docs](https://docs.microsoft.com/en-us/microsoftteams/platform/)


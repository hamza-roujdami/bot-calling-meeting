# Voice Features Guide

This bot has **speech-to-text** and **text-to-speech** capabilities powered by Azure AI Services.

## 🎤 **Current Voice Capabilities**

### **What the Bot CAN Do:**

✅ **Record voice messages** during calls  
✅ **Transcribe speech to text** (Azure Speech Service)  
✅ **Convert text to speech** and play announcements  
✅ **Play pre-recorded audio prompts**  
✅ **Download and process call recordings**  

### **What the Bot CANNOT Do (Currently):**

❌ **Real-time wake word detection** ("Hey Bot", "Mr Bot")  
❌ **Continuous listening** during calls  
❌ **Bidirectional voice conversation** (like Alexa/Siri)  

**Why?** The bot uses **service-hosted media** (Microsoft manages the audio streams). For real-time voice interaction, you would need **application-hosted media** (much more complex).

---

## 🎯 **How Voice Interaction Works**

### **Current Workflow:**

```
1. Bot joins meeting
   ↓
2. Bot plays prompt: "Please record your message"
   ↓
3. User speaks
   ↓
4. Bot records audio
   ↓
5. Bot transcribes using Azure Speech Service
   ↓
6. Bot posts to meeting chat: "You said: [transcribed text]"
```

### **Control Method:**

You control the bot through **chat messages** in the meeting, not voice commands:

| **Chat Command** | **Bot Action** |
|-----------------|----------------|
| `hello` | Greets you by name + shows welcome card |
| `playrecordprompt` | Starts voice recording |
| `joinscheduledmeeting` | Joins the current meeting |
| Click "Play record prompt" button | Bot prompts for voice message |
| Click "Hang up" button | Bot leaves the call |

---

## 🔧 **Enabled Features with Azure Speech Services**

### **1. Speech-to-Text Transcription**

When you record a voice message:
- Bot downloads the recording
- Sends to Azure Speech Service
- Gets back text transcription
- Posts to chat: "You said: [your message]"

**Code**: `CallingBot.cs` lines 158-195

### **2. Text-to-Speech Announcements**

When a user joins an incident call:
- Bot converts text to speech: "There is an ongoing incident for [subject]. Your assistance is required."
- Plays the audio to all participants

**Code**: `CallingBot.cs` lines 255-268

### **3. Audio Prompts**

Bot can play:
- `speech.wav` - Generic speech prompt
- `please-record-your-message.wav` - Recording prompt
- **Dynamic TTS** - Generated from any text

---

## 🧪 **How to Test Voice Features**

### **Test 1: Voice Recording & Transcription**

1. **In the meeting chat**, send: `playrecordprompt`
2. **Bot will play**: "Please record your message"
3. **Speak clearly**: "This is a test message"
4. **Bot will transcribe** and post: "You said: This is a test message"

### **Test 2: Create Incident with Voice Announcement**

1. **In personal chat**, send: `hello`
2. **Click**: "Create Incident" button
3. **Fill in**:
   - Subject: "Network Outage"
   - Select participants
4. **Submit**
5. **Bot will**:
   - Create meeting
   - Join it
   - Invite participants
   - Play voice announcement when they join

### **Test 3: Meeting Controls**

While bot is in the meeting:

| **Button** | **Action** |
|-----------|-----------|
| **Play record prompt** | Bot prompts for voice message |
| **Invite participant** | Add someone to the meeting |
| **Transfer call** | Transfer to another user |
| **Hang up** | Bot leaves |

---

## 🔑 **Configuration**

Your Speech Service is configured:

```json
"CognitiveServices": {
  "Enabled": true,
  "SpeechKey": "Bxx3eeh...ACOGxuoC",
  "SpeechRegion": "eastus2",
  "SpeechRecognitionLanguage": "en-US"
}
```

**Endpoints:**
- Speech-to-Text: `https://eastus2.stt.speech.microsoft.com`
- Text-to-Speech: `https://eastus2.tts.speech.microsoft.com`
- Cognitive Services: `https://voiceagentdevfoundry1513.cognitiveservices.azure.com/`

---

## 💡 **Interaction Model**

### **Current: Chat-Controlled Bot**

```
You (in chat): "playrecordprompt"
   ↓
Bot (voice): "Please record your message"
   ↓
You (speak): "What do you think about this proposal?"
   ↓
Bot (in chat): "You said: What do you think about this proposal?"
```

### **Not Supported: Voice-Activated Bot**

```
❌ You (during call): "Hey Bot, what do you think?"
❌ Bot (voice): "I think it's a great idea!"
```

**Why?** This requires:
- Application-hosted media (bot manages audio streams)
- Real-time speech recognition
- Natural language understanding
- Voice synthesis and streaming
- Much more complex implementation

---

## 🚀 **What You Can Do Right Now**

### **In Your Current Meeting:**

1. **Send chat message**: `playrecordprompt`
2. **Wait for bot's voice prompt**
3. **Speak your message**
4. **Bot transcribes and shows in chat**

### **For Incident Calls:**

1. **Create incident** with subject
2. **Bot joins and announces**: "There is an ongoing incident for [subject]"
3. **Participants join** and hear the announcement
4. **Bot can record** participant responses

---

## 📊 **Voice Features Summary**

| **Feature** | **Status** | **How to Use** |
|------------|------------|----------------|
| Speech-to-Text | ✅ Enabled | Click "Play record prompt" button |
| Text-to-Speech | ✅ Enabled | Automatic on incident calls |
| Audio Prompts | ✅ Working | Pre-recorded WAV files |
| Wake Word Detection | ❌ Not Supported | Use chat commands instead |
| Real-time Voice Chat | ❌ Not Supported | Service-hosted media limitation |
| Voice Transcription | ✅ Enabled | After recording completes |

---

## 🎯 **Best Use Cases**

**This bot is perfect for:**
- 📞 Managing conference calls via chat
- 🎙️ Recording voice messages and transcribing them
- 📢 Playing announcements to participants
- 👥 Inviting/removing participants
- 📝 Converting meeting voice notes to text
- 🚨 Creating incident response calls

**This bot is NOT designed for:**
- 🗣️ Real-time voice conversation
- 🎤 Voice-activated commands
- 💬 Voice-only interactions

---

## 🔮 **Future Enhancements (If Needed)**

To add wake word detection and voice responses:

1. **Switch to Application-Hosted Media**
   - Bot receives raw audio streams
   - Much more complex to implement
   - Requires media processing infrastructure

2. **Implement Wake Word Detection**
   - Use Azure Speech SDK with keyword recognition
   - Listen for "Hey Bot", "Mr Bot", etc.

3. **Add Natural Language Processing**
   - Use Azure OpenAI or Language Understanding (LUIS)
   - Process voice commands
   - Generate contextual responses

4. **Real-time Voice Synthesis**
   - Stream TTS responses back to call
   - Handle audio mixing with other participants

**Estimated effort**: 2-3 weeks of development

---

## 📚 **Resources**

- [Azure Speech Service Documentation](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/)
- [Microsoft Graph Calling API](https://docs.microsoft.com/en-us/graph/api/resources/calls-api-overview)
- [Teams Calling Bots Overview](https://docs.microsoft.com/en-us/microsoftteams/platform/bots/calls-and-meetings/calls-meetings-bots-overview)
- [Service-Hosted vs Application-Hosted Media](https://docs.microsoft.com/en-us/microsoftteams/platform/bots/calls-and-meetings/requirements-considerations-application-hosted-media-bots)

---

## 🎉 **Summary**

**Your bot NOW has:**
- ✅ Azure Speech Services **ENABLED**
- ✅ Speech-to-text transcription
- ✅ Text-to-speech announcements
- ✅ Personalized greetings by name
- ✅ Meeting join and control capabilities

**Test it in your current meeting by sending: `playrecordprompt`** 🎤


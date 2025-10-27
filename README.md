# Remote Camera Plugin for Claude Code

Capture and analyze photos from your mobile device directly in Claude Code. Perfect for hardware debugging, PCB inspection, chip identification, and any task where Claude needs to see something in the physical world.

## Features

- **Remote photo capture**: Generate URLs that open a camera interface on mobile devices
- **Hardware debugging**: Inspect circuit boards, components, and electronic devices
- **Chip identification**: Read and identify chip markings, part numbers, and labels
- **Visual analysis**: Analyze error screens, handwritten notes, schematics, and documentation
- **Reusable capture sessions**: Take multiple photos without generating new URLs
- **Integrated workflow**: Photos automatically available for Claude to analyze

## Installation

### Prerequisites

1. **S3-compatible storage**: You'll need access to an S3-compatible bucket (AWS S3, Backblaze B2, etc.)
2. **Claude Code**: Latest version installed

### Step 1: Set Up S3 Storage and Generate Auth Token

Visit **[https://www.ai.moda/mcp-servers/remote-camera/](https://www.ai.moda/mcp-servers/remote-camera/)** for complete setup instructions.

The setup guide will walk you through:
- Creating an S3-compatible storage bucket (AWS S3, Backblaze B2, or other providers)
- Configuring the required permissions and CORS settings
- Generating your authentication token

At the end of the setup, you'll receive a base64-encoded token that looks like:
```
eyJzM1VybCI6Imh0dHBzOi8vczMudXMtZWFzdC0wMDUuYmFja2JsYXplYjIuY29tL3lvdXItYnVja2V0LW5hbWUvIiwiYWNjZXNzS2V5SWQiOiJ5b3VyLWFjY2Vzcy1rZXktaWQiLCJzZWNyZXRBY2Nlc3NLZXkiOiJ5b3VyLXNlY3JldC1hY2Nlc3Mta2V5IiwicmVnaW9uIjoidXMtZWFzdC0wMDUifQ==
```

Copy this token for the next step.

### Step 2: Configure Environment Variable

Set the `REMOTE_CAMERA_AUTH_TOKEN` environment variable with your base64-encoded token:

**macOS/Linux (in ~/.zshrc or ~/.bashrc)**:
```bash
export REMOTE_CAMERA_AUTH_TOKEN="eyJzM1VybCI6Imh0dHBzOi8v..."
```

**Or set it per-session**:
```bash
export REMOTE_CAMERA_AUTH_TOKEN="eyJzM1VybCI6Imh0dHBzOi8v..."
claude
```

**Windows (PowerShell)**:
```powershell
$env:REMOTE_CAMERA_AUTH_TOKEN = "eyJzM1VybCI6Imh0dHBzOi8v..."
```

### Step 3: Install Plugin

From the ai.moda marketplace:
```
/plugin marketplace add aimoda/claude-code-plugin-marketplace
/plugin install remote-camera@aimoda
```

For local development:
1. Clone or copy this plugin to your local machine
2. Add the plugin directory to your Claude Code settings
3. Restart Claude Code

## Usage

Once installed, Claude automatically uses the remote camera skill when you need to capture photos from the physical world.

### Basic Usage

Simply describe what you need help with:

```
"Can you identify this chip on my board?"
```

Claude will:
1. Generate a capture URL with instructions
2. Present the URL for you to open on your mobile device
3. Wait for you to take and upload the photo
4. Automatically analyze the photo once uploaded

### Example Workflows

**Hardware Debugging**:
```
User: "This circuit isn't working, can you help debug it?"

Claude: I'll help you debug the circuit. Let me generate a remote camera
capture link so I can see your board...

[capture URL provided]

Please open this URL on your mobile device and take a photo showing your
full PCB with all components visible. I'll wait for the upload...

[analyzes photo after upload]

I can see the issue - the polarized capacitor C3 is installed backwards...
```

**Chip Identification**:
```
User: "What chip is this?"

Claude: Let me take a look. I've generated a capture link...

[provides URL with instruction: "Take a close-up photo of the chip markings"]

[waits for upload and analyzes]

This is a Texas Instruments TPS54331 step-down converter...
```

**Multiple Photos**:
```
User: "I need help with several components on my board."

Claude: [generates capture URL]

After you upload the first photo, you can use the same URL to take
additional photos. I'll detect each new upload automatically.
```

### Manual Invocation

While Claude automatically uses this skill when appropriate, you can also explicitly request photo capture:

- "Take a photo of..."
- "Can you see..."
- "Help me identify..."
- "Show me what's wrong with..."

## How It Works

### Capture Workflow

1. **Generate URL**: Claude calls `generate_upload_url` with clear instructions
2. **User captures**: You open the URL on your mobile device and take a photo
3. **Upload**: The photo uploads directly to your S3 bucket
4. **Poll**: Claude polls your S3 bucket waiting for the upload
5. **Analyze**: Once uploaded, Claude downloads and analyzes the photo

### Technical Details

- **Upload URLs**: Valid for 7 days, reusable for multiple captures
- **Download URLs**: Valid for 7 days
- **Storage**: Photos stored in your S3 bucket at `remote-camera-mcp/{session-id}`
- **Session IDs**: UUID format, serves as the filename
- **File format**: Any image format supported by your mobile device (JPEG, HEIC, PNG, etc.)

### Privacy & Security

- **Your storage**: All photos stored in your S3 bucket, you control the data
- **Private URLs**: Presigned URLs with limited validity (7 days)
- **No server storage**: The MCP server doesn't store any photos, only facilitates upload/download
- **Credentials**: Your S3 credentials never leave your machine (sent only in Authorization header)

## Use Cases

### Hardware & Electronics

- PCB debugging and inspection
- Component identification
- Chip part number reading
- Solder joint inspection
- Trace layout verification
- Connector pin mapping
- Wire color identification

### Documentation

- Handwritten notes transcription
- Schematic diagram analysis
- Datasheet page capture
- Reference design review
- Error message screenshots

### Troubleshooting

- Error screen capture
- Status display reading
- Diagnostic information
- Assembly verification
- Configuration checking

## Troubleshooting

### "Missing Authorization header" Error

**Cause**: Environment variable not set or not accessible to Claude Code

**Solution**:
1. Verify `REMOTE_CAMERA_AUTH_TOKEN` is set: `echo $REMOTE_CAMERA_AUTH_TOKEN`
2. Restart Claude Code after setting the environment variable
3. Ensure Claude Code was started from a shell that has the variable set

### "Invalid base64 token" Error

**Cause**: Auth token is not properly base64-encoded

**Solution**:
1. Regenerate the token at https://www.ai.moda/mcp-servers/remote-camera/
2. Copy the complete token without any extra whitespace or newlines
3. Verify the token decodes to valid JSON: `echo $REMOTE_CAMERA_AUTH_TOKEN | base64 -d`

### "Token must contain valid s3Url string" Error

**Cause**: S3 credentials JSON is missing required fields or has incorrect format

**Solution**:
1. Regenerate your token at https://www.ai.moda/mcp-servers/remote-camera/ ensuring all S3 settings are correct
2. Verify your credentials JSON has all required fields: `s3Url`, `accessKeyId`, `secretAccessKey`, `region`
3. Ensure `s3Url` uses HTTPS protocol and ends with a forward slash `/`

### Poll Timeout

**Cause**: Photo wasn't uploaded within the timeout period (default 120 seconds)

**Solution**:
1. Ensure you opened the capture URL on your mobile device
2. Check your mobile device has internet connectivity
3. Try uploading again - the URL is still valid
4. Check CORS configuration on your S3 bucket

### Photo Not Clear

**Cause**: Lighting, focus, or angle issues

**Solution**:
1. Claude will guide you on what would help
2. Use the same capture URL to take a better photo
3. Ensure good lighting on the subject
4. Get close enough to see details clearly
5. Hold the device steady

## Configuration

### Custom Timeout

The default polling timeout is 120 seconds (2 minutes). Claude adjusts this based on the task, but you can request longer waits:

```
"Take your time, I'll wait up to 5 minutes for the photo"
```

### Storage Location

Photos are stored at: `s3://your-bucket/remote-camera-mcp/{session-id}`

You can clean up old photos periodically or set up S3 lifecycle policies to auto-delete after a certain period.

### Security Best Practices

1. **Bucket isolation**: Use a dedicated bucket for remote camera captures
2. **Credential rotation**: Rotate S3 credentials periodically
3. **Lifecycle policies**: Auto-delete old captures after 7-30 days
4. **Access logging**: Enable S3 access logging for audit trail
5. **Encryption**: Enable server-side encryption on your bucket

## Support & Resources

- **Setup Guide**: https://www.ai.moda/mcp-servers/remote-camera/
- **MCP Server Repository**: [remote-camera-mcp](https://github.com/your-org/remote-camera-mcp)
- **Plugin Issues**: File issues on this repository
- **Claude Code Documentation**: https://docs.claude.com/

## License

MIT License - See LICENSE file for details

## Contributing

Contributions welcome! Please file issues and pull requests on the repository.

---

**Security Notice**: Never commit your S3 credentials or auth token to version control. Always use environment variables for sensitive credentials.

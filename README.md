# N8N UGC Video Generation Workflow

This repository contains a working n8n workflow for generating UGC (User Generated Content) videos using AI. The workflow takes a video prompt and two images (model avatar and product image) to create a video that transitions between them.

## Workflow Overview

The **UGC Video Generation Workflow** creates professional-looking videos by:
- Starting with a model avatar image (first frame)
- Ending with a product image (last frame)
- Generating smooth transitions based on your video prompt
- Using AI-powered video generation via Replicate API

## Features

- **Webhook Trigger**: Easy integration with external applications
- **Smart Polling**: Automatically checks generation status until completion
- **Error Handling**: Gracefully handles failures with detailed error messages
- **Flexible Input**: Accepts custom prompts and image URLs
- **Production Ready**: Includes proper response handling and status tracking

## Workflow Structure

```
Webhook Trigger
    ↓
Extract Input Data
    ↓
Start Video Generation (Replicate API)
    ↓
Extract Prediction Info
    ↓
Wait Before Polling (3 seconds)
    ↓
Check Generation Status
    ↓
Check If Complete? ──[Yes]──→ Format Success Response → Send Response
    ↓                                                           ↑
   [No]                                                        │
    ↓                                                          │
Check If Failed? ──[Yes]──→ Format Error Response ────────────┘
    ↓
   [No]
    ↓
Loop Back to Wait (Continue Polling)
```

## Setup Instructions

### Prerequisites

1. **n8n Installation**: Have n8n installed and running
   - Self-hosted: https://docs.n8n.io/hosting/
   - Cloud: https://n8n.io/cloud/

2. **Replicate API Account**:
   - Sign up at https://replicate.com/
   - Get your API token from https://replicate.com/account/api-tokens

### Installation Steps

1. **Import the Workflow**:
   - Open your n8n instance
   - Go to **Workflows** → **Add Workflow** → **Import from File**
   - Select `workflows/ugc-video-generation.json`

2. **Configure Replicate API Credentials**:
   - Go to **Credentials** in n8n
   - Create new **Header Auth** credential
   - Set header name: `Authorization`
   - Set header value: `Token YOUR_REPLICATE_API_TOKEN`
   - Save as "Replicate API"

3. **Assign Credentials to HTTP Request Nodes**:
   - Open the workflow
   - Click on "Start Video Generation (Replicate)" node
   - Under **Authentication**, select your "Replicate API" credential
   - Repeat for "Check Generation Status" node
   - Save the workflow

4. **Activate the Workflow**:
   - Toggle the workflow to **Active**
   - Copy the webhook URL from the "Webhook Trigger" node

## Usage

### API Request Format

Send a POST request to your webhook URL with the following JSON body:

```json
{
  "video_prompt": "A smooth transition showcasing the product features with professional lighting",
  "model_avatar_url": "https://example.com/model-avatar.jpg",
  "product_image_url": "https://example.com/product-image.jpg"
}
```

### Example using cURL

```bash
curl -X POST https://your-n8n-instance.com/webhook/ugc-video-generate \
  -H "Content-Type: application/json" \
  -d '{
    "video_prompt": "Smooth transition from model to product with fade effect",
    "model_avatar_url": "https://images.unsplash.com/photo-1494790108377-be9c29b29330",
    "product_image_url": "https://images.unsplash.com/photo-1523275335684-37898b6baf30"
  }'
```

### Example using JavaScript

```javascript
const response = await fetch('https://your-n8n-instance.com/webhook/ugc-video-generate', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    video_prompt: 'Dynamic product reveal with smooth camera movement',
    model_avatar_url: 'https://example.com/model.jpg',
    product_image_url: 'https://example.com/product.jpg'
  })
});

const data = await response.json();
console.log('Video URL:', data.videoUrl);
```

### Example using Python

```python
import requests

url = 'https://your-n8n-instance.com/webhook/ugc-video-generate'
payload = {
    'video_prompt': 'Professional product demonstration with elegant transitions',
    'model_avatar_url': 'https://example.com/model.jpg',
    'product_image_url': 'https://example.com/product.jpg'
}

response = requests.post(url, json=payload)
result = response.json()
print(f"Video URL: {result['videoUrl']}")
```

## Response Format

### Success Response

```json
{
  "videoUrl": "https://replicate.delivery/pbxt/xyz123.mp4",
  "status": "succeeded",
  "message": "Video generated successfully!"
}
```

### Error Response

```json
{
  "error": "Error message details",
  "status": "failed"
}
```

## Input Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `video_prompt` | string | Yes | Description of the video transition and style |
| `model_avatar_url` | string | Yes | URL to the model/avatar image (starting frame) |
| `product_image_url` | string | Yes | URL to the product image (ending frame) |

## Video Generation Settings

The workflow uses the following default settings for Stable Video Diffusion:

- **Motion Bucket ID**: 127 (controls motion intensity)
- **FPS**: 24 (frames per second)
- **Number of Frames**: 60 (approximately 2.5 seconds at 24 fps)
- **Conditioning Augmentation**: 0.02 (stability factor)

You can modify these in the "Start Video Generation" node if needed.

## Customization Options

### Change Video Duration

Edit the `num_frames` parameter in the "Start Video Generation" node:
- 24 frames = 1 second @ 24fps
- 60 frames = 2.5 seconds @ 24fps
- 120 frames = 5 seconds @ 24fps

### Adjust Motion Intensity

Modify the `motion_bucket_id` parameter:
- Lower values (50-100): Subtle motion
- Medium values (100-150): Moderate motion
- Higher values (150-255): Intense motion

### Change Polling Interval

Edit the "Wait Before Polling" node to change how often status is checked:
- Default: 3 seconds
- For shorter videos: 2 seconds
- For longer videos: 5-10 seconds

## Alternative Video Generation APIs

While this workflow uses Replicate's Stable Video Diffusion model, you can adapt it for other services:

### HeyGen API
Replace the Replicate nodes with HeyGen API calls for avatar-based videos:
- API: https://docs.heygen.com/

### RunwayML Gen-2/Gen-3
For more advanced video generation:
- API: https://docs.runwayml.com/

### D-ID
For talking avatar videos:
- API: https://docs.d-id.com/

## Troubleshooting

### Common Issues

1. **"Authentication failed"**
   - Verify your Replicate API token is correct
   - Ensure credentials are properly assigned to HTTP Request nodes

2. **"Video generation timeout"**
   - Increase polling interval
   - Check Replicate API status
   - Verify image URLs are accessible

3. **"Invalid image URL"**
   - Ensure URLs are publicly accessible
   - Check image format (JPG, PNG supported)
   - Verify URLs use HTTPS protocol

4. **"Workflow doesn't start"**
   - Ensure workflow is activated (toggle on)
   - Check webhook URL is correct
   - Verify n8n instance is running

### Debug Mode

Enable debug mode in n8n settings to see detailed execution logs:
1. Go to **Settings** → **Log output**
2. Set to **Verbose**
3. Check execution logs for detailed error messages

## Performance Considerations

- **Video Generation Time**: Typically 30-60 seconds per video
- **Concurrent Requests**: Depends on Replicate API limits
- **Image Size**: Optimal resolution is 1024x576 or 576x1024
- **Timeout**: Default workflow timeout is 5 minutes

## Cost Estimation

Replicate pricing (as of 2026):
- Stable Video Diffusion: ~$0.03-0.05 per video
- Check current pricing: https://replicate.com/pricing

## Advanced Features

### Add Webhook Authentication

Modify the "Webhook Trigger" node to require authentication:
1. Add header authentication
2. Validate API keys
3. Implement rate limiting

### Store Generated Videos

Add nodes to:
- Upload to AWS S3
- Save to Google Drive
- Store in database

### Queue Management

For high-volume usage:
- Add queue system (Redis, RabbitMQ)
- Implement priority queuing
- Add retry logic

## Contributing

Feel free to enhance this workflow:
- Add support for more video generation models
- Implement batch processing
- Add webhook notifications on completion
- Create UI for parameter adjustment

## License

This workflow is provided as-is for use with n8n and Replicate API.

## Support

For issues and questions:
- n8n Documentation: https://docs.n8n.io/
- Replicate Documentation: https://replicate.com/docs
- Community Forum: https://community.n8n.io/

## Version History

- **v1.0** (2026-01-06): Initial release with Stable Video Diffusion support

---

**Note**: Always test with sample images before processing production content. Monitor API usage to avoid unexpected costs.

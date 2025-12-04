# Kai.ai Media Generator

Build a versatile AI media generator powered by Kai.ai APIs. Create images with GPT-4o and generate videos using Veo, Runway, Sora, or Kling models. Enter prompts, configure settings, generate media, and view your creations in a unified gallery.

## Prerequisites

**IMPORTANT**: Before starting, read the Kai.ai documentation:
```bash
Read ai_docs/kie api's/image/4o Image API Quickstart.md
Read ai_docs/kie api's/video/Veo 3.1 AI Video(Fast&Quality).md
Read ai_docs/kie api's/video/Runway API Quickstart.md
```

Focus on the text-to-image, text-to-video, and image-to-video generation examples.

**Environment Setup**:
- Validate that you have access to a Kai.ai API key - check the root `.env` file for `KIE_API_KEY` without exposing it
- The backend will load this key using `python-dotenv`

## Core Features

### 1. Image Generation (GPT-4o)
- Large text area for prompt input
- Aspect ratio dropdown: 1:1 (Square), 3:2 (Landscape), 2:3 (Portrait)
- Variant count selector: 1, 2, or 4 images
- Prompt enhancement toggle (isEnhance)
- Generate button with loading spinner and progress indicator
- Display all generated images in a grid
- Download button for each image

### 2. Video Generation
- Large text area for prompt input
- Model selector dropdown:
  - Veo 3.1 Fast (veo3_fast) - Quick generation
  - Veo 3.1 Quality (veo3) - Higher quality
  - Runway (720p/1080p) - Reliable generation
  - Sora 2 - OpenAI's text-to-video
  - Kling 2.5 Turbo - Image-to-video specialist
- Aspect ratio dropdown: 16:9 (Landscape), 9:16 (Portrait)
- Duration selector: 5s or 10s (model dependent)
- Optional reference image URL for image-to-video
- Generate button with loading spinner and status updates
- Display generated video with playback controls
- Download button for video

### 3. Media Gallery
- Tabbed view: All / Images / Videos
- Grid view of all previously generated media
- Show prompt text on hover
- Click to view full-size (images) or play (videos)
- Delete individual items
- Clear all media button
- Filter by model type
- Sort by newest/oldest

### 4. Settings
- API key input (saved to database)
- Default image model preference
- Default video model preference
- Default aspect ratios for image/video
- Default variant/duration settings
- Save/reset settings

## Database Schema

```sql
-- Store generated media with metadata
CREATE TABLE media (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    media_type TEXT NOT NULL CHECK (media_type IN ('image', 'video')),
    prompt TEXT NOT NULL,
    model TEXT NOT NULL,
    aspect_ratio TEXT NOT NULL,
    task_id TEXT,
    filename TEXT NOT NULL,
    file_path TEXT NOT NULL,
    thumbnail_path TEXT,
    duration INTEGER,
    status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'processing', 'completed', 'failed')),
    error_message TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    completed_at DATETIME
);

-- User settings
CREATE TABLE settings (
    id INTEGER PRIMARY KEY CHECK (id = 1),
    api_key TEXT,
    default_image_model TEXT DEFAULT 'gpt4o-image',
    default_video_model TEXT DEFAULT 'veo3_fast',
    default_image_aspect_ratio TEXT DEFAULT '1:1',
    default_video_aspect_ratio TEXT DEFAULT '16:9',
    default_variant_count INTEGER DEFAULT 1,
    default_video_duration INTEGER DEFAULT 5,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Task queue for async processing
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    media_id INTEGER NOT NULL,
    task_id TEXT NOT NULL,
    status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'processing', 'completed', 'failed')),
    progress REAL DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (media_id) REFERENCES media(id) ON DELETE CASCADE
);
```

## Backend API

**Image Generation:**
- `POST /api/images/generate` - Generate images from prompt
  - Body: `{ prompt, aspect_ratio, n_variants, is_enhance }`
  - Returns: `{ task_id, media_id }`

**Video Generation:**
- `POST /api/videos/generate` - Generate video from prompt
  - Body: `{ prompt, model, aspect_ratio, duration, image_url? }`
  - Returns: `{ task_id, media_id }`

**Task Status:**
- `GET /api/tasks/{task_id}` - Get task status and progress
  - Returns: `{ status, progress, result_urls?, error? }`

**Media:**
- `GET /api/media` - List all media with optional filtering
  - Query params: `type`, `model`, `sort_by`
- `GET /api/media/{id}` - Get single media metadata
- `DELETE /api/media/{id}` - Delete media item
- `DELETE /api/media/all` - Clear all media

**Settings:**
- `GET /api/settings` - Get current settings
- `PUT /api/settings` - Update settings

**Static Files:**
- `GET /static/media/{filename}` - Serve generated media files

## Frontend Structure

**Pages:**
1. **Home/Generate** (/)
   - Tab selector: Images | Videos
   - Prompt input area
   - Configuration controls (model, aspect ratio, variants/duration)
   - Generate button
   - Results display area with progress tracking

2. **Gallery** (/gallery)
   - Tab filter: All | Images | Videos
   - Grid of all generated media
   - Filter by model
   - Sort controls
   - Delete actions

3. **Settings** (/settings)
   - API key input
   - Default preferences for images
   - Default preferences for videos
   - Save button

**Components:**
- PromptInput (textarea with character count)
- MediaTypeSelector (tabs for image/video)
- ImageConfigPanel (aspect ratio, variants, enhance toggle)
- VideoConfigPanel (model, aspect ratio, duration, reference image)
- MediaGrid (responsive grid of images/videos)
- ImageCard (thumbnail with prompt overlay and actions)
- VideoCard (thumbnail with play button, duration, and actions)
- MediaModal (full-size image viewer or video player)
- ProgressIndicator (task progress with status updates)
- SettingsForm (configuration inputs)

## Key Implementation Details

**Backend:**
1. Install `requests`, `python-dotenv`, and `pillow`
2. Store API key in environment variable or database
3. Save generated media to `/static/media/` directory
4. Generate unique filenames: `{type}_{timestamp}_{uuid}.{ext}`
5. Implement async task polling for generation status
6. Download remote media files and store locally
7. Generate video thumbnails if possible
8. Handle multiple image variants in single request

**Frontend:**
1. Install `axios` for API requests
2. Create Pinia stores for media and settings state
3. Implement polling for task status during generation
4. Media lazy loading in gallery
5. Form validation for prompt input
6. Loading states with progress indicators
7. Toast notifications for success/errors
8. Video player with standard controls

**Image API Integration Example:**
```python
import requests

def generate_image(prompt: str, aspect_ratio: str, n_variants: int, is_enhance: bool, api_key: str):
    url = "https://api.kie.ai/api/v1/gpt4o-image/generate"
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }
    payload = {
        "prompt": prompt,
        "size": aspect_ratio,  # "1:1", "3:2", or "2:3"
        "nVariants": n_variants,  # 1, 2, or 4
        "isEnhance": is_enhance
    }

    response = requests.post(url, json=payload, headers=headers)
    result = response.json()

    if response.ok and result.get('code') == 200:
        return result['data']['taskId']
    raise Exception(result.get('msg', 'Generation failed'))

def check_image_status(task_id: str, api_key: str):
    url = f"https://api.kie.ai/api/v1/gpt4o-image/record-info?taskId={task_id}"
    headers = {"Authorization": f"Bearer {api_key}"}

    response = requests.get(url, headers=headers)
    result = response.json()

    if response.ok and result.get('code') == 200:
        data = result['data']
        return {
            'status': 'completed' if data['successFlag'] == 1 else
                     'failed' if data['successFlag'] == 2 else 'processing',
            'progress': float(data.get('progress', 0)),
            'result_urls': data.get('response', {}).get('result_urls', []),
            'error': data.get('errorMessage')
        }
    raise Exception(result.get('msg', 'Status check failed'))
```

**Video API Integration Example (Veo 3.1):**
```python
import requests

def generate_video(prompt: str, model: str, aspect_ratio: str, image_urls: list = None, api_key: str = None):
    url = "https://api.kie.ai/api/v1/veo/generate"
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }
    payload = {
        "prompt": prompt,
        "model": model,  # "veo3" or "veo3_fast"
        "aspectRatio": aspect_ratio,  # "16:9" or "9:16"
        "enableTranslation": True
    }

    if image_urls:
        payload["imageUrls"] = image_urls
        payload["generationType"] = "FIRST_AND_LAST_FRAMES_2_VIDEO"

    response = requests.post(url, json=payload, headers=headers)
    result = response.json()

    if response.ok and result.get('code') == 200:
        return result['data']['taskId']
    raise Exception(result.get('msg', 'Generation failed'))

def check_video_status(task_id: str, api_key: str):
    # Similar polling pattern as image API
    # Returns status, progress, and result URLs when complete
    pass
```

**Runway Video API Integration:**
```python
def generate_runway_video(prompt: str, duration: int, quality: str, aspect_ratio: str, image_url: str = None, api_key: str = None):
    url = "https://api.kie.ai/api/v1/runway/generate"
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }
    payload = {
        "prompt": prompt,
        "duration": duration,  # 5 or 10
        "quality": quality,  # "720p" or "1080p" (1080p only for 5s)
        "aspectRatio": aspect_ratio,  # "16:9", "9:16", "1:1", "4:3", "3:4"
        "waterMark": ""
    }

    if image_url:
        payload["imageUrl"] = image_url

    response = requests.post(url, json=payload, headers=headers)
    result = response.json()

    if response.ok and result.get('code') == 200:
        return result['data']['taskId']
    raise Exception(result.get('msg', 'Generation failed'))
```

## UI Design
Build a modern, professional interface with:
- Clean, minimal design with proper spacing
- Modern color palette (primary/secondary/accent colors)
- Responsive layout that works on mobile and desktop
- Professional typography (readable fonts, clear hierarchy)
- Smooth transitions and hover effects
- Consistent component styling throughout
- Clear distinction between image and video generation modes
- Progress indicators with percentage and status text
- Proper form validation with user feedback
- Loading states for async operations
- Video player with full playback controls

## Success Criteria

- Generate 1-4 images from a single prompt using GPT-4o
- Generate videos using multiple model options (Veo, Runway, Sora, Kling)
- Configure aspect ratio and quality settings
- Track generation progress with real-time status updates
- View all generated media in organized gallery with tabs
- Filter media by type and model
- Download individual images and videos
- Delete unwanted media
- Support image-to-video generation with reference images
- Save API key and default preferences
- Media properly served from backend
- Fast, responsive UI with good UX
- Clear error messages for API failures
- Proper handling of async task polling

**Build time: ~30-40 minutes**

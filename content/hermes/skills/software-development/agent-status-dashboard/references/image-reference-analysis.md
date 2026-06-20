# Handling User Reference Images from imgbb

When user provides an `ibb.co` URL (e.g. "ทำแบบในรูป https://ibb.co/XXXXXX"):

## Step 1: Find the real image URL

Navigate to the imgbb page, find the "Full image (linked)" section. The direct image URL is typically:
`https://i.ibb.co/XXXXX/filename.jpg`

Two URLs per page:
- Thumbnail: `https://i.ibb.co/XXXX/thumb.jpg` (smaller)
- Full image: `https://i.ibb.co/XXXXX/full.jpg` (larger, 2400×1080)

Always get the full image for analysis.

## Step 2: Download and analyze

```bash
curl -s -o /tmp/ref.jpg "https://i.ibb.co/XXXXX/filename.jpg"
```

```python
from PIL import Image
img = Image.open('/tmp/ref.jpg')
w, h = img.size
```

## Step 3: Color band analysis

Sample horizontal bands to understand the layout:

```python
# For each horizontal stripe (every 30-40px), compute:
# - Average RGB
# - Count of specific elements: wood/beige (floor), blue (walls), 
#   white (background), dark (text), green/red (status indicators)

for y_start in range(0, h, 40):
    r_sum, g_sum, b_sum = 0, 0, 0
    wood, white, dark, green, red = 0, 0, 0, 0, 0
    for y in range(y_start, min(y_start+40, h), 4):
        for x in range(0, w, 4):
            r, g, b = img.getpixel((x, y))
            # Identify element types by color ranges
            if 120 < r < 210 and 80 < g < 170: wood += 1
            if r > 200 and g > 200 and b > 200: white += 1
            if r < 60 and g < 60 and b < 60: dark += 1
            if g > r+20 and g > b+20: green += 1
            if r > g+20 and r > b+20 and r > 150: red += 1
    print(f'y={y_start}: wood={wood}% white={white}% text={dark}% green={green}% red={red}%')
```

## Step 4: Interpret the layout

- **White-dominant bands** (= header/content areas): Clean UI with light background
- **Dark pixel bands** (= text columns): Left/right margins with labels or data
- **Red/green bands** (= status indicators): Agent status, buttons, or alert elements
- **Wood/beige bands** (= could be pixel art floor): Warns of game-style UI — verify if user wants that

## Step 5: Build matching UI

For clean light-theme UIs (white-dominant images):
- `background: #f0f2f5` for page
- White cards with subtle borders
- Status badges: green for working, gray for idle
- Clean sans-serif fonts (Inter, Prompt)
- Responsive grid layout
- Auto-refresh every 15s

DO NOT build pixel art or canvas-based interfaces from a light-theme reference image.
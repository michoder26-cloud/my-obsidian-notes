# BBR Warm Palette — Exact Measurements

Extracted from image analysis of the BrewBerich Co., Ltd. (BBR) pixel agent dashboard reference design. Image: 640×288px progressive JPEG.

## Layout Dimensions

| Section | Width | Height | Position |
|---------|-------|--------|----------|
| Pixel Office (left) | ~62% (398px) | 100% | x=0..398 |
| Vertical divider | 2px at x=395 | 100% | x=395..397 |
| Status Panel (right) | ~38% (242px) | 100% | x=398..640 |
| Navigation header | 100% | ~32px | y=0..32 |

Left panel: Pixel office = 62% width. Right panel: Terminal/status = 38% width.

## Color Swatches

### Page & Navigation
| Sample | Region | RGB | Hex |
|--------|--------|-----|-----|
| Nav background | top bar | `rgb(61,50,36)` | `#3d3224` |
| Nav link text | header links | `rgb(200,184,136)` | `#c8b888` |
| Page background | below nav | `rgb(232,224,200)` | `#e8e0c8` |

### Pixel Office Floor & Walls
| Sample | Region | RGB | Hex |
|--------|--------|-----|-----|
| Wall/edge | x=5, y=5 | `rgb(23,20,15)` | `#17140f` |
| Floor tile dark | x=100, y=100 | `rgb(121,96,32)` | `#796020` |
| Floor tile light | grid center | `rgb(138,122,74)` | `#8a7a4a` |
| Grid line | between tiles | `rgba(0,0,0,0.1)` | — |

### Desk & Furniture
| Sample | Region | RGB | Hex |
|--------|--------|-----|-----|
| Desk top | x=170, y=140 | `rgb(149,138,106)` | `#958a6a` |
| Desk highlight | top edge | `rgb(168,154,120)` | `#a89a78` |
| Desk side/leg | x=200, y=155 | `rgb(209,206,197)` | `#d1cec5` |
| Chair | x=180, y=165 | `rgb(211,187,123)` | `#d3bb7b` |
| Keyboard | center of desk | `rgb(106,90,58)` | `#6a5a3a` |

### Monitor
| Sample | Region | RGB | Hex |
|--------|--------|-----|-----|
| Monitor screen | x=170, y=130 | `rgb(56,186,114)` | `#38ba72` |
| Monitor bezel | around screen | `rgb(34,34,34)` | `#222` |
| Monitor inner | screen edge | `rgb(51,51,51)` | `#333` |

### Potted Plant
| Sample | Region | RGB | Hex |
|--------|--------|-----|-----|
| Pot | x=50, y=150 | `rgb(58,42,26)` | `#3a2a1a` |
| Leaves dark | top of plant | `rgb(74,122,58)` | `#4a7a3a` |
| Leaves light | leaf highlight | `rgb(90,154,74)` | `#5a9a4a` |

### Pixel Characters
| Sample | Region | RGB | Hex |
|--------|--------|-----|-----|
| Agent body | x=120, y=130 | `rgb(232,208,138)` | `#e8d08a` |
| Agent head | x=120, y=115 | `rgb(170,143,54)` | `#aa8f36` |
| Agent eyes | face area | `rgb(255,255,255)` | `#fff` |

### Speech Bubble
| Sample | Region | RGB | Hex |
|--------|--------|-----|-----|
| Bubble background | x=160, y=100 | `rgba(255,255,240,0.95)` | `#fffff0` |
| Bubble border | bubble edge | `rgb(160,144,112)` | `#a09070` |
| Text (name) | inside bubble | `rgb(61,50,36)` | `#3d3224` |
| Text (role) | inside bubble | `rgb(122,106,74)` | `#7a6a4a` |

### Divider
| Sample | Region | RGB | Hex |
|--------|--------|-----|-----|
| Office/panel divider | x=395 | `rgb(163,116,72)` | `#a37448` |
| Room divider (office) | inner column | `rgb(163,116,72)` | `#a37448` |

### Terminal Status Panel
| Sample | Region | RGB | Hex |
|--------|--------|-----|-----|
| Panel header | x=430, y=10 | `rgb(70,53,45)` | `#46352d` |
| Panel background | x=430, y=60 | `rgb(160,120,68)` | `#a07844` |
| Panel body | x=450, y=60 | `rgb(200,184,136)` | `#c8b888` |
| Panel bottom | x=430, y=250 | `rgb(151,103,57)` | `#976739` |
| List item BG | middle of list | `rgb(185,182,167)` | `#b9b6a7` |
| Terminal text | x=455, y=60 | `rgb(2,24,12)` | `#02180c` |

### Agent Status Tags
| Sample | Region | RGB | Hex |
|--------|--------|-----|-----|
| "Working" tag bg | status badge | `rgb(86,184,70)` | `#56b846` |
| "Idle" tag bg | status badge | `rgb(122,106,74)` | `#7a6a4a` |

## Layout Grid (Canvas Top-Down Office)

The office canvas uses a cell grid where each cell = 48px. Room is 25 cells wide × 16 cells tall.

### Room Zones
- **Zone 1** (columns 0..5): 2 desks at rows 1 and 4
- **Zone 2** (columns 6..11): 2 desks at rows 1 and 4
- **Zone 3** (columns 12..17): 2 desks at rows 1 and 4

### Desk Layout
Each desk occupies 3 cells wide × 2 cells tall (144×96px):
- Desk body: fills 3×2 cell grid
- Monitor: sits 6px above desk top, 24px wide × 18px tall
- Keyboard: at bottom of desk
- Chair: below desk, 14×14px centered
- Name label: below chair at +26px
- Potted plant: at desk x=0 and x=5 (far left/far right corners)

### Divider Positions
- Between zone 1 & 2: at x = CELL × 5.5 = 264px
- Between zone 2 & 3: at x = CELL × 11 = 528px

### Character Position
- Standing position: centered on desk x + CELL × 1.5, y = desk.y + CELL + 25
- Walk target: random in office area (x: 40..W-80, y: 30..290)

## Character Sprite Dimensions

| Part | Width | Height | Offset |
|------|-------|--------|--------|
| Shadow | 24px ellipse | 8px tall | at y + 20 |
| Legs (each) | 4px | 6px + legOff | at body bottom |
| Body | 14px | 14px | at y + 1 |
| Head | 12px | 7px | at y - 4 |
| Eyes (each) | 3px | 2px | at y - 1 |
| Level badge | 6px | 4px | at body + 16 |
| Emoji indicator | 10px font | — | at y - 12 |
| Speech bubble | 56px | 18px | at y - 32 |

## Dashboard Metric Cards

- 4 cards in a row using CSS Grid `grid-template-columns: repeat(4, 1fr)`
- Card background: `#fff8ee`, border: `#d8c8a0`, radius: 6px
- Numbers use `#a37448` (orange-brown) by default, `#56b846` for wins, `#c0392b` for losses, `#2c6b9e` for avg R:R
- Labels: 11px, `#7a6a4a`

## Navigation Bar

- Height: 32-40px, background: `#3d3224`
- Border bottom: 2px solid `#a37448`
- Links: `color: #c8b888`, hover: `background: rgba(200,184,136,0.1); color: #e8d8a8`
- Brand/text: `color: #e8d8a8; font-weight: bold; letter-spacing: 1px`
- Live dot: 7px circle, `#56b846` when on, `#7a6a4a` when off

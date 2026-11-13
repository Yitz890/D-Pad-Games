# D-Pad Games — How to Install & Create Games

## Installing Games

### From the app
1. Go to **Settings → Games → Add a game**
2. Pick a `.zip` file from your phone
3. Done — the game appears in your Games list

### From a computer (ADB)
1. Connect your phone via USB
2. Run this command:
```
adb push mygame.zip /sdcard/Android/data/com.yitz.dpadgames/files/games/
```
3. Open or switch to D-Pad Games — it auto-installs the ZIP

> ADB install replaces existing games with the same ID automatically.

### Removing a game
Settings → Games → highlight the game → press OK → Remove

---

## Creating Your Own Game

### What you need
- A text editor
- A web browser (for testing)
- Basic HTML/JavaScript knowledge

### Step 1 — Create your files

At minimum you need two files:

**game.json** — tells the app about your game:
```json
{
  "id": "my-cool-game",
  "name": "My Cool Game",
  "description": "A fun game I made",
  "entry": "index.html",
  "version": "1.0"
}
```

**index.html** — your game:
```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>My Cool Game</title>
  <style>
    body { margin: 0; background: #111; color: #fff; font-family: monospace; }
    #msg { text-align: center; padding-top: 40%; font-size: 1.4rem; }
  </style>
</head>
<body>
  <div id="msg">Press OK to start!</div>
  <script>
    // Listen for D-pad buttons
    window.addEventListener('dpadkey', function(e) {
      var key = e.detail.key;     // 'up','down','left','right','select','back','pause'
      var phase = e.detail.phase; // 'down' or 'up'

      if (phase === 'down') {
        document.getElementById('msg').textContent = 'You pressed: ' + key;
      }
    });
  </script>
</body>
</html>
```

### Step 2 — ZIP it up

Select your files (not the folder) and compress them into a `.zip`:

```
my-cool-game/
  game.json
  index.html
```

> Important: the ZIP should contain the files at the root, not inside a subfolder.

### Step 3 — Install and play

Use either install method above. Your game shows up in the Games list!

---

## D-Pad Controls

Your game receives button presses as `dpadkey` events:

```javascript
window.addEventListener('dpadkey', function(e) {
  if (e.detail.phase === 'down') {
    var key = e.detail.key;
    if (key === 'up')     { /* move up */ }
    if (key === 'down')   { /* move down */ }
    if (key === 'left')   { /* move left */ }
    if (key === 'right')  { /* move right */ }
    if (key === 'select') { /* OK / confirm */ }
    if (key === 'pause')  { /* pause the game */ }
    if (key === 'back')   { /* go back */ }
  }
});
```

The number keys also work: 2=up, 4=left, 5=select, 6=right, 8=pause.

---

## Adding Settings

Create a `settings.json` to give players options. The app shows these automatically in its settings screen.

```json
[
  {
    "key": "speed",
    "title": "Speed",
    "description": "How fast the game runs",
    "type": "choice",
    "choices": ["Slow", "Normal", "Fast"],
    "default": 1
  },
  {
    "key": "sound",
    "title": "Sound",
    "description": "Play sound effects",
    "type": "toggle",
    "default": true
  }
]
```

Read settings in your game:
```javascript
window.addEventListener('dpadsettings', function(e) {
  if (e.detail) {
    var speed = e.detail.speed;   // 0, 1, or 2
    var sound = e.detail.sound;   // true or false
  }
});
```

---

## Saving High Scores

Each game gets its own `localStorage`:

```javascript
// Save
localStorage.setItem('best', String(score));

// Load
var best = parseInt(localStorage.getItem('best'), 10) || 0;
```

---

## Simple Sound Effects

```javascript
function beep(freq, duration) {
  try {
    var ctx = new (window.AudioContext || window.webkitAudioContext)();
    var o = ctx.createOscillator();
    var g = ctx.createGain();
    o.type = 'square';
    o.frequency.value = freq;
    g.gain.setValueAtTime(0.05, ctx.currentTime);
    g.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + duration);
    o.connect(g);
    g.connect(ctx.destination);
    o.start();
    o.stop(ctx.currentTime + duration);
  } catch (e) {}
}

beep(440, 0.1);   // short bleep
beep(880, 0.3);   // higher, longer
```

---

## Rules to Follow

- **ES5 JavaScript only** — no `let`, `const`, `=>`, or template strings
- **Old CSS only** — no `flex gap`, `inset`, `aspect-ratio`, or `backdrop-filter`
- **No internet** — everything must be in your ZIP, no CDN links
- **Screen is 240×320** — use `rem` units or percentages so it scales
- **Max 100 files**, 24MB total, 8MB per file
- **Allowed file types**: `.html` `.js` `.css` `.json` `.png` `.jpg` `.gif` `.svg` `.webp` `.mp3` `.ogg` `.wav` `.ttf` `.woff` `.woff2` `.txt` `.md`

---

## Testing on Desktop

Your game works in a regular browser too. Add this so arrow keys work during testing:

```javascript
document.addEventListener('keydown', function(e) {
  var map = {
    ArrowUp: 'up', ArrowDown: 'down',
    ArrowLeft: 'left', ArrowRight: 'right',
    Enter: 'select', Escape: 'back'
  };
  if (map[e.key]) {
    e.preventDefault();
    window.dispatchEvent(new CustomEvent('dpadkey', {
      detail: { key: map[e.key], phase: 'down' }
    }));
  }
});
```

---

## game.json Rules

| Field         | Rules                                              |
|---------------|----------------------------------------------------|
| `id`          | 3-40 chars, lowercase, letters/numbers/dash/underscore, must start with a letter |
| `name`        | 1-32 characters                                    |
| `description` | Up to 100 characters                               |
| `entry`       | Path to your HTML file (must end in `.html`)        |
| `version`     | Version string (like `"1.0"`)                      |

---

That's it! Start simple, test in a browser, ZIP it, install it. Have fun!

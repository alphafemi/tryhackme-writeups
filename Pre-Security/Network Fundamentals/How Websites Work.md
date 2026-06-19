## Web Fundamentals & Basic Security Issues

---

## 1. How Websites Work

### The Request-Response Cycle

```
[Browser]  ───── Request (e.g. "give me this page") ────►  [Web Server]
[Browser]  ◄──── Response (HTML, CSS, JS data) ───────────  [Web Server]
   │
   ▼
Renders the page for you
```

A **web server** is simply a dedicated computer somewhere that handles your requests and sends back data.

---

### Two Major Components of a Website

| Component | Also Known As | Role |
|---|---|---|
| **Front End** | Client-Side | How your browser **renders** the website |
| **Back End** | Server-Side | **Processes** requests and returns responses |

---
---

## 2. The Three Building Blocks

```
┌─────────────────────────────────┐
│           HTML                  │  → Structure
│  ┌───────────────────────────┐  │
│  │           CSS              │ │  → Styling/Appearance
│  │  ┌──────────────────────┐ │  │
│  │  │     JavaScript        │ │ │  → Interactivity/Behaviour
│  │  └──────────────────────┘ │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

| Language | Purpose |
|---|---|
| **HTML** | Builds the website and defines its **structure** |
| **CSS** | Makes the website **look good** (styling) |
| **JavaScript** | Adds **interactivity** and complex features |

---
---

## 3. HTML (HyperText Markup Language)

### Basic HTML Structure
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Page Title</title>
  </head>
  <body>
    <h1>This is a heading</h1>
    <p>This is a paragraph</p>
  </body>
</html>
```

### Structure Breakdown

| Tag | Purpose |
|---|---|
| `<!DOCTYPE html>` | Declares this is an HTML5 document |
| `<html>` | The root element — everything else sits inside this |
| `<head>` | Contains page info (e.g. title) — not shown on the page |
| `<body>` | Contains all **visible** content |
| `<h1>` | A large heading |
| `<p>` | A paragraph |

> Other common tags: `<button>`, `<img>`, lists, and many more

---

### Attributes
Attributes add extra information/behaviour to elements.

```html
<p class="bold-text">Hello</p>
<img src="img/cat.jpg">
<p attribute1="value1" attribute2="value2">Text</p>
```

| Attribute | Purpose | Uniqueness |
|---|---|---|
| **class** | Used for styling | Can be **shared** by multiple elements |
| **id** | Identifies a specific element | Must be **unique** per page |
| **src** | Specifies file location (e.g. images, scripts) | — |

> 💡 You can view any website's HTML via **"View Page Source"** (Chrome) or **"Show Page Source"** (Safari)

---
---

## 4. JavaScript (JS)

### What Does JavaScript Do?
While HTML builds the **structure**, JavaScript controls the **functionality** — making pages interactive and dynamic.

### Including JavaScript

**Inline (within the page):**
```html
<script>
  document.getElementById("demo").innerHTML = "Hack the Planet";
</script>
```

**External file:**
```html
<script src="/location/of/javascript_file.js"></script>
```

### Example — Changing Content
```javascript
document.getElementById("demo").innerHTML = "Hack the Planet";
```
> Finds the element with `id="demo"` and changes its text

### Example — Event Handling
```html
<button onclick='document.getElementById("demo").innerHTML = "Button Clicked";'>
  Click Me!
</button>
```

| Event | Triggers When... |
|---|---|
| **onclick** | An element is clicked |
| **onhover** | An element is hovered over |

---
---

## 5. Basic Security Issues

### 🔓 Sensitive Data Exposure

**What it is:**
When a website **fails to remove** sensitive clear-text information from its frontend code (HTML/JavaScript).

```
[Page Source Code]
     │
     ├── <!-- Temp login: admin / password123 --> ❌ Exposed!
     │
[Attacker views source] ──► Finds credentials ──► Uses them to log in
```

**Examples of exposed data:**
- Login credentials left in HTML comments
- Hidden links to private areas of the site
- API keys or tokens in JavaScript files

> 🔍 **Security Tip:** Always check a website's page source code first when assessing for vulnerabilities — exposed credentials or hidden links are often easy wins for attackers.

---

### 💉 HTML Injection

**What it is:**
A vulnerability where **unfiltered user input** is displayed directly on a page, allowing attackers to inject their own HTML code.

```
[User Input Field] ──► No Sanitisation ──► Input rendered as HTML on the page
                                                    │
                                          Attacker injects malicious HTML
```

**Why it happens:**
- The website fails to **sanitise** (filter/clean) user input
- That unsanitised input gets directly used/displayed on the page

> ⚠️ Related but different: **Database Injection** — manipulating database queries via input (covered separately) — this is **server-side**, while HTML Injection is **client-side**.

---
---


**Correct screen: creating the Service Portal**

Fill the fields like this:

| Field                 | What to enter                                                             |
| --------------------- | ------------------------------------------------------------------------- |
| **Title**             | `DTS Service Portal`                                                      |
| **URL suffix**        | `dts`                                                                     |
| **Homepage**          | Leave empty for now                                                       |
| **Application**       | Keep `Campus Asset` or select the application where you want this project |
| **Theme**             | Leave empty for now                                                       |
| **Main menu**         | Leave empty                                                               |
| **Catalog home page** | Leave empty                                                               |
| **KB home page**      | Leave empty                                                               |

Then click **Submit** at the top right.

### After clicking Submit
You should see your portal in the list. **Do not worry about Homepage yet.**

-----
**Next: create the homepage.**

## Step 2 — Create `DTS Home` page

In the left **All** navigator, search:
**Service Portal → Pages**
Then click **New**.
Fill only these:

| Field     | Value      |
| --------- | ---------- |
| **Title** | `DTS Home` |
| **ID**    | `dts_home` |

Then click **Submit**.

### After submitting
Go back to:
**Service Portal → Portals**
Open your **DTS Service Portal** and set:
**Homepage → DTS Home**
Then click **Update**.
After that, we will start **Step 3: creating the exact red top header** like in your screenshot.

---

Great 👍 Now we start building the **actual UI**.

# Step 3 — Create the top red header

We will create this part first:

```text
┌──────────────────────────────────────────────┐
│  YOUR LOGO                          LOG IN   │
├──────────────────────────────────────────────┤
│  DIGITAL TECHNOLOGY SOLUTIONS                │
└──────────────────────────────────────────────┘
```

## 3.1 Create a Widget

Go to:
**All → Service Portal → Widgets**
Click **New**.
Set:

**Name:**

```text
DTS Top Header
```

### HTML Template

Paste:

```html
<div class="dts-header">

  <div class="top-bar">

    <div class="logo">
      <span class="logo-icon">◉</span>
      <span class="logo-text">JUMBO</span>
    </div>

    <div class="header-links">

      <a href="javascript:void(0)">
        ❤ SERVICE STATUS
      </a>

      <a href="javascript:void(0)">
        ⇥ LOG IN
      </a>

    </div>

  </div>

  <div class="sub-bar">
    DIGITAL TECHNOLOGY SOLUTIONS
  </div>

</div>
```

### CSS

Paste:

```css
.dts-header {
  width: 100%;
  font-family: Arial, sans-serif;
}

.top-bar {
  height: 62px;
  background: #e60028;

  display: flex;
  align-items: center;
  justify-content: space-between;

  padding: 0 8%;
}

.logo {
  display: flex;
  align-items: center;
  color: white;
}

.logo-icon {
  font-size: 25px;
  margin-right: 10px;
}

.logo-text {
  font-size: 25px;
  font-weight: 500;
  letter-spacing: 1px;
}

.header-links {
  display: flex;
  gap: 30px;
}

.header-links a {
  color: white;
  text-decoration: none;
  font-size: 12px;
  font-weight: bold;
}

.sub-bar {
  background: #c90020;
  color: white;

  height: 41px;

  display: flex;
  align-items: center;

  padding-left: 8%;

  font-size: 16px;
  font-weight: bold;

  letter-spacing: 0.5px;
}
```

You can leave **Server Script** and **Client Controller** empty for now.
Click **Submit**.

---

# Step 4 — Add Header to `DTS Home`

Go to:
**Service Portal → Page Designer**
Open:

```text
DTS Home
```

Create a new **Row → 12 columns**.
Drag your widget:

```text
DTS Top Header
```

into that row.
Then **Save** and open:

```text
https://YOUR-INSTANCE.service-now.com/dts
```

You should now see your **red top header + DIGITAL TECHNOLOGY SOLUTIONS bar**.
After this, say **next**. Then we'll create the **exact hero section with background image, "DTS Service Portal" title, and large search bar**.

<img width="1143" height="107" alt="image" src="https://github.com/user-attachments/assets/9bce8f47-9056-4488-8751-51f48149860f" />

---
Great. **Next we build the Hero section** — the large background area with:

* `DTS Service Portal`
* `Guest, how can we help?`
* Search button

# Step 5 — Create Hero Widget

Go to:
**All → Service Portal → Widgets → New**
Set:
**Name:** `DTS Hero Section`
## HTML Template

Paste:

```html
<div class="hero-section">

  <div class="hero-overlay">

    <h1>DTS Service Portal</h1>

    <div class="portal-search">

      <input
        type="text"
        placeholder="Guest, how can we help?"
        ng-model="c.searchText"
        ng-keypress="c.checkEnter($event)">

      <button
        type="button"
        ng-click="c.search()"
        aria-label="Search">

        <span class="glyphicon glyphicon-search"></span>

      </button>

    </div>

  </div>

</div>
```

## CSS

Paste this:

```css
.hero-section {
  min-height: 330px;
  width: 100%;

  background-image: url("YOUR_IMAGE_URL");
  background-size: cover;
  background-position: center;

  position: relative;
}

.hero-overlay {
  min-height: 330px;
  width: 100%;

  background: rgba(255, 255, 255, 0.58);

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;

  padding: 40px 20px;
}

.hero-overlay h1 {
  margin: 0 0 20px;

  color: #333;
  font-size: 48px;
  font-weight: 500;
}

.portal-search {
  width: 70%;
  max-width: 735px;

  height: 48px;

  display: flex;

  background: #fff;
  border: 1px solid #aaa;
  border-radius: 4px;
}

.portal-search input {
  flex: 1;

  height: 100%;

  border: none;
  outline: none;

  padding: 0 15px;

  font-size: 16px;
  color: #333;
}

.portal-search button {
  width: 55px;
  height: 100%;

  background: #fff;

  border: none;
  border-left: 1px solid #ccc;

  font-size: 18px;

  cursor: pointer;
}

.portal-search button:hover {
  background: #f5f5f5;
}
```

## Client Controller

Paste:

```javascript
api.controller = function() {
  var c = this;

  c.search = function() {
    if (c.searchText) {
      alert("Searching for: " + c.searchText);
    }
  };

  c.checkEnter = function(event) {
    if (event.which === 13) {
      c.search();
    }
  };
};
```

For now, leave **Server Script** empty.

---

# Step 6 — Add the Hero to the Homepage

Go to:
**Service Portal → Page Designer**
Open:

**DTS Home**

Add a new:

```text
Row
└── 12 Columns
    └── DTS Hero Section
```

Place this row **directly below your DTS Top Header**.

## Background image

For now, `YOUR_IMAGE_URL` will not work until we upload an image.
You can first test without the image by replacing:

```css
background-image: url("YOUR_IMAGE_URL");
```

with:

```css
background: #e8e8e8;
```

After you confirm the title and search bar are showing correctly, the next step is to upload a background image into ServiceNow and make it look much closer to the Cincinnati portal screenshot.
Then we'll build the **4 exact cards: Report an Issue, Request Something, Software Directory, Manage Password**.

<img width="1132" height="282" alt="image" src="https://github.com/user-attachments/assets/2fdbaddd-49e8-46f8-a4a9-e96dcaa7be5a" />

---
Perfect 👍 Your **Hero section is working correctly**. Now we build the **next section: the 4 cards**, like the UC website.

We will do this **one step at a time**.

# Step 7 — Create the 4 Service Cards widget

The result will look like:

```text
┌──────────────┬──────────────┬──────────────┬──────────────┐
│      🎧      │      🌐      │      📁      │      🔒      │
│              │              │              │              │
│ Report an    │ Request      │ Software     │ Manage       │
│ Issue        │ Something    │ Directory    │ Password     │
│              │              │              │              │
│ Contact IT   │ Browse the   │ Browse and   │ Reset your   │
│ Support      │ catalog      │ request      │ password     │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

## 7.1 Create a new widget

Go to:

**All → Service Portal → Widgets**

Click **New**.

### Name

```text
DTS Service Cards
```

## 7.2 HTML Template

Paste:

```html
<div class="service-cards">

  <div class="service-card">
    <div class="card-icon">
      <span class="glyphicon glyphicon-headphones"></span>
    </div>

    <h3>Report an Issue</h3>

    <p>Contact IT support to report an issue</p>
  </div>


  <div class="service-card">
    <div class="card-icon">
      <span class="glyphicon glyphicon-globe"></span>
    </div>

    <h3>Request Something</h3>

    <p>Browse the catalog to request a service</p>
  </div>


  <div class="service-card">
    <div class="card-icon">
      <span class="glyphicon glyphicon-folder-open"></span>
    </div>

    <h3>Software Directory</h3>

    <p>Browse and request software</p>
  </div>


  <div class="service-card">
    <div class="card-icon">
      <span class="glyphicon glyphicon-lock"></span>
    </div>

    <h3>Manage Password</h3>

    <p>Set up or reset your password</p>
  </div>

</div>
```

## 7.3 CSS

Paste:

```css
.service-cards {
  display: flex;
  width: 86%;
  max-width: 1200px;
  margin: -40px auto 40px;
  position: relative;
  z-index: 10;
}

.service-card {
  flex: 1;
  min-height: 175px;

  background: #ffffff;
  border: 1px solid #d4d4d4;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;

  text-align: center;
  padding: 20px;

  box-shadow: 0 2px 5px rgba(0,0,0,0.12);
}

.service-card:hover {
  background: #f7f7f7;
  cursor: pointer;
}

.card-icon {
  font-size: 38px;
  margin-bottom: 12px;
  color: #333;
}

.service-card h3 {
  font-size: 17px;
  font-weight: 500;
  color: #333;
  margin: 0 0 10px;
}

.service-card p {
  font-size: 13px;
  color: #777;
  margin: 0;
  line-height: 1.4;
}

/* Mobile */
@media (max-width: 768px) {
  .service-cards {
    flex-direction: column;
    margin-top: 0;
  }
}
```

Leave **Client Controller** and **Server Script** empty.

Click **Submit**.

**Stop after creating this widget.** Once the `DTS Service Cards` widget is created, say **done**. Then I'll show you exactly how to add a **12-column layout below the Hero section** and place these cards there.
<img width="1119" height="254" alt="image" src="https://github.com/user-attachments/assets/bddb9b8b-04a0-4791-812a-4696d241613b" />

---
Perfect 👍 The **Hero + 4 cards** are now working. It already looks very close to the UC portal.

Next, we will build the section below the cards:

# Step 8 — Create the IT Service Desk box

We will create this:

```text
┌─────────────────────────────────┐
│ IT Service Desk Hours            │
│ ─────────────────────────────── │
│ ● The Service Desk is currently │
│   open                           │
│                                  │
│ Monday–Thursday: 7 a.m.–7 p.m.   │
│ Friday: 7 a.m.–5 p.m.            │
│                                  │
│ ☎ 513-555-4357                   │
│ ◉ Service Desk Support Options   │
└─────────────────────────────────┘
```

## Step 8.1 — Create a new widget

Go to:

**All → Service Portal → Widgets → New**

### Name

```text
DTS Service Desk
```

## Step 8.2 — HTML Template

Paste:

```html
<div class="service-desk">

  <div class="desk-title">
    IT Service Desk Hours
  </div>

  <div class="desk-content">

    <div class="status">
      <span class="status-dot"></span>
      <strong>The Service Desk is currently open</strong>
    </div>

    <p class="hours">
      Monday-Thursday, 7 a.m.-7 p.m.; Friday, 7 a.m.-5 p.m.
    </p>

    <p>
      Contact IT support during regular operating hours, or report an issue anytime.
    </p>

    <div class="contact">
      <span class="glyphicon glyphicon-earphone"></span>
      <a href="javascript:void(0)">513-555-4357</a>
    </div>

    <div class="contact">
      <span class="glyphicon glyphicon-globe"></span>
      <a href="javascript:void(0)">Service Desk Support Options</a>
    </div>

    <small>
      Online requests can be submitted at any time.
    </small>

  </div>

</div>
```

## Step 8.3 — CSS

Paste:

```css
.service-desk {
  width: 290px;
  margin: 25px auto 80px;
  background: #fff;
  border: 1px solid #cfcfcf;
  border-radius: 3px;
  color: #555;
  font-size: 13px;
}

.desk-title {
  padding: 12px 15px;
  font-size: 16px;
  color: #444;
  border-bottom: 3px solid #d6002a;
}

.desk-content {
  padding: 12px 15px 15px;
}

.status {
  margin-bottom: 12px;
  font-size: 12px;
}

.status-dot {
  display: inline-block;
  width: 8px;
  height: 8px;
  background: #2e9b58;
  border-radius: 50%;
  margin-right: 7px;
}

.hours {
  font-weight: bold;
  font-size: 12px;
}

.desk-content p {
  line-height: 1.5;
  margin: 8px 0;
}

.contact {
  margin: 9px 0;
}

.contact span {
  width: 20px;
}

.contact a {
  color: #b00020;
  text-decoration: underline;
  margin-left: 5px;
}

.desk-content small {
  color: #777;
}
```

Leave **Server Script** and **Client Controller** empty.

Click **Submit**.

**Stop after creating this widget.** Then say **done**. Next, I'll show you exactly how to add it below the 4 cards.
<img width="1056" height="373" alt="image" src="https://github.com/user-attachments/assets/69b4b80a-838d-47b5-bde3-e41228bcc7c6" />

---

Perfect 👍 Next we build the **footer**, the last major section needed to match the UC portal.

# Step 9 — Create the Footer Widget

Go to:

**All → Service Portal → Widgets → New**

### Name

```text
DTS Footer
```

## 9.1 HTML Template

Paste:

```html
<div class="portal-footer">

  <!-- Left black section -->
  <div class="footer-left">

    <div class="social-links">
      <a href="javascript:void(0)">
        <span class="glyphicon glyphicon-thumbs-up"></span>
      </a>

      <a href="javascript:void(0)">𝕏</a>

      <a href="javascript:void(0)">
        <span class="glyphicon glyphicon-camera"></span>
      </a>

      <a href="javascript:void(0)">
        <span class="glyphicon glyphicon-play"></span>
      </a>

      <a href="javascript:void(0)">in</a>
    </div>

    <div class="address">
      <strong>DTS</strong><br>
      Your Organization<br>
      India<br>
      +91 00000 00000
    </div>

  </div>


  <!-- Right red section -->
  <div class="footer-right">

    <div class="footer-column">
      <h4>DTS Tools</h4>

      <a href="javascript:void(0)">Service Catalog</a>
      <a href="javascript:void(0)">Knowledge</a>
      <a href="javascript:void(0)">Requests</a>
      <a href="javascript:void(0)">IT Help</a>
      <a href="javascript:void(0)">Support</a>
    </div>

    <div class="footer-column">
      <h4>About Us</h4>

      <a href="javascript:void(0)">About</a>
      <a href="javascript:void(0)">Services</a>
      <a href="javascript:void(0)">Contact</a>
      <a href="javascript:void(0)">Privacy Policy</a>
      <a href="javascript:void(0)">Terms</a>
    </div>

  </div>

</div>
```

## 9.2 CSS

Paste:

```css
.portal-footer {
  width: 100%;
  min-height: 240px;

  display: flex;

  margin-top: 0;
  color: white;
}

/* LEFT */
.footer-left {
  width: 30%;
  min-height: 240px;

  background: #080808;

  padding: 35px 50px;
}

.social-links {
  display: flex;
  gap: 20px;

  margin-bottom: 30px;
}

.social-links a {
  color: white;
  font-size: 20px;

  text-decoration: none;
}

.address {
  color: #ddd;
  font-size: 13px;
  line-height: 1.7;
}

/* RIGHT */
.footer-right {
  flex: 1;

  background: #e60028;

  display: flex;

  gap: 100px;

  padding: 35px 70px;
}

.footer-column {
  display: flex;
  flex-direction: column;
}

.footer-column h4 {
  font-size: 16px;
  margin: 0 0 15px;
  color: white;
}

.footer-column a {
  color: white;

  text-decoration: none;

  font-size: 13px;

  margin-bottom: 9px;
}

.footer-column a:hover {
  text-decoration: underline;
}

/* MOBILE */
@media (max-width: 768px) {

  .portal-footer {
    flex-direction: column;
  }

  .footer-left {
    width: 100%;
  }

  .footer-right {
    gap: 40px;
    padding: 30px;
  }
}
```

Leave **Server Script** and **Client Controller** empty.

Click **Submit**.

**Stop here after creating the widget.** Say **done**, then I'll give you the next exact step to add it below the Service Desk section.
<img width="1142" height="241" alt="image" src="https://github.com/user-attachments/assets/748ae775-767d-4728-b99a-1879bcdb44f7" />

---
**Widgets:**
<img width="1044" height="537" alt="image" src="https://github.com/user-attachments/assets/27cf89eb-4064-42a2-bbfb-e75414494c28" />

**Pages:**
<img width="762" height="472" alt="image" src="https://github.com/user-attachments/assets/c4b17d07-56f1-47bc-badc-21a572ad8400" />

**Output:**
<img width="1366" height="1360" alt="image" src="https://github.com/user-attachments/assets/8a5569f1-a0f9-4b7f-be27-3d1c4376fb33" />

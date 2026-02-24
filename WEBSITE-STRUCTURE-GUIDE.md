# Flight Crew ID - Website Structure Guide

## 📁 File Organization

Your website now consists of the following files:

```
flight-crew-id/
│
├── index.html                          # Home page (main landing page)
├── about.html                          # About us page
├── contact.html                        # Contact form page
├── application.html                    # Application redirect page
├── flight-crew-application-payment.html # Full payment form (4-step application)
│
├── styles.css                          # Main stylesheet (shared by all pages)
├── main.js                            # Main JavaScript (shared by all pages)
│
└── (Additional pages to create:)
    ├── benefits.html
    ├── testimonials.html
    ├── aviation.html
    ├── drone.html
    ├── login.html
    ├── faq.html
    ├── terms.html
    └── news.html
```

## 🔗 How Pages Are Connected

### Navigation Menu (On Every Page)
All pages have the same header navigation:
- **Home** → `index.html`
- **About** → `about.html`
- **News** → `news.html`
- **Benefits** → `benefits.html`
- **Testimonials** → `testimonials.html`
- **Contact** → `contact.html`
- **Login** → `login.html`

### Call-to-Action Buttons
From **index.html** (home page):
- "Apply Now" buttons → `application.html` → auto-redirects to `flight-crew-application-payment.html`
- "Learn More" for Aviation → `aviation.html`
- "Learn More" for Drones → `drone.html`

### Footer Links (On Every Page)
Quick Links column:
- Home → `index.html`
- About → `about.html`
- Terms & Conditions → `terms.html`
- Contact → `contact.html`
- FAQ → `faq.html`

Services column:
- General Aviation → `aviation.html`
- Drone Operators → `drone.html`
- Benefits → `benefits.html`
- Testimonials → `testimonials.html`

## 🎨 Shared Resources

### styles.css
This single CSS file contains ALL the styles for the entire website:
- Header & Navigation
- Hero Slider
- Service Cards
- Forms
- Testimonials
- Footer
- Mobile Responsive Rules

**To change the color scheme**, edit these values in `styles.css`:
```css
/* Primary blue color */
#0066cc → Change to your color

/* Dark background */
#1a1a1a → Change to your color

/* Hover states */
#0052a3 → Change to your color
```

### main.js
This JavaScript file handles:
- Mobile hamburger menu toggle
- Hero slider auto-advance
- Smooth scrolling
- Active page highlighting in navigation

## 📄 Page Descriptions

### 1. index.html (Home Page)
- **Purpose**: Main landing page
- **Features**: 
  - Auto-rotating hero slider
  - Service cards for Aviation & Drone crews
  - Apply Now buttons
- **Links to**: application.html, aviation.html, drone.html

### 2. about.html
- **Purpose**: Company information
- **Features**: 
  - Mission statement
  - List of who we serve
  - Quality commitment
- **Links to**: application.html

### 3. contact.html
- **Purpose**: Contact form
- **Features**: 
  - Contact form with validation
  - Contact information
  - Business hours
- **Form Action**: Currently logs to console (needs backend integration)

### 4. application.html
- **Purpose**: Redirect page to application
- **Features**: 
  - Auto-redirects after 2 seconds
  - Manual "Continue" button
- **Redirects to**: flight-crew-application-payment.html

### 5. flight-crew-application-payment.html
- **Purpose**: Full 4-step application with payment
- **Features**: 
  - Step 1: Account creation
  - Step 2: Product selection & add-ons
  - Step 3: Document upload
  - Step 4: Stripe payment
  - Success confirmation
- **Note**: Standalone page with its own styles (embedded)

## 🚀 How to Test Locally

### Option 1: Double-Click (Easiest)
1. Put all files in the same folder
2. Double-click `index.html`
3. Click navigation links to test all pages

### Option 2: VS Code Live Server (Best for Editing)
1. Open folder in VS Code
2. Install "Live Server" extension
3. Right-click `index.html` → "Open with Live Server"
4. Edit files and see changes instantly

### Option 3: Python Server
```bash
cd your-website-folder
python -m http.server 8000
# Visit: http://localhost:8000
```

## 📱 Mobile Menu

The mobile hamburger menu:
- Appears on screens < 768px wide
- Click to toggle menu open/close
- Automatically closes when clicking a link
- Defined in `styles.css` (@media query)
- Controlled by `main.js`

## 🔧 Customization Guide

### Change Logo
In each HTML file, find:
```html
<a href="index.html" class="logo">
    <span>✈️</span> FLIGHT CREW ID
</a>
```
Replace the emoji with `<img src="your-logo.png" alt="Logo">` or change the text.

### Change Colors
Edit `styles.css`:
```css
/* Find and replace these colors */
#0066cc  /* Primary blue */
#1a1a1a  /* Dark header/footer */
#f5f5f5  /* Light backgrounds */
```

### Add New Page
1. Copy any existing HTML file
2. Rename it (e.g., `benefits.html`)
3. Change the `<title>` tag
4. Change the active class in navigation
5. Add your content in the `<main>` section
6. All styling and navigation will work automatically!

### Change Hero Images
In `index.html`, find the slider sections and replace:
```html
style="background-image: linear-gradient(...)"
```
with:
```html
style="background-image: url('your-image.jpg')"
```

## 🎯 Testing Checklist

- [ ] All navigation links work
- [ ] Mobile menu opens and closes
- [ ] Hero slider auto-advances
- [ ] Forms validate properly
- [ ] "Apply Now" buttons redirect correctly
- [ ] Footer links work
- [ ] Page looks good on mobile (resize browser)
- [ ] Active page is highlighted in navigation

## 🐛 Common Issues

**Problem**: Navigation doesn't highlight active page
- **Solution**: Make sure the page filename matches exactly (case-sensitive)

**Problem**: Styles don't load
- **Solution**: Ensure `styles.css` is in the same folder as HTML files

**Problem**: JavaScript doesn't work
- **Solution**: Ensure `main.js` is in the same folder as HTML files

**Problem**: Payment form looks different
- **Solution**: Normal! The payment form has embedded styles for standalone use

**Problem**: Links go to 404
- **Solution**: Create the missing page files (benefits.html, etc.)

## 📦 Production Deployment

When ready to deploy:

1. **Create missing pages** (benefits.html, testimonials.html, etc.)
2. **Add real images** (replace placeholder image URLs)
3. **Set up backend** for contact form
4. **Configure Stripe** with real API keys
5. **Get a domain name**
6. **Choose a host**:
   - Netlify (free for static sites)
   - Vercel (free for static sites)
   - GitHub Pages (free)
   - Traditional hosting (Bluehost, HostGator, etc.)

## 🔐 Security Notes

- Never put Stripe secret keys in frontend code
- Use HTTPS for production
- Validate all form inputs on server-side
- Implement CSRF protection for forms

## 💡 Tips

1. **Consistent Navigation**: Every page has the same header/footer
2. **Shared Styles**: One CSS file = easy to update entire site
3. **Mobile-First**: Site is fully responsive
4. **Easy to Expand**: Copy any page to create new ones
5. **SEO-Friendly**: Each page has unique title tag

## 📞 Next Steps

1. Create the remaining pages listed in the file structure
2. Add real content to each page
3. Replace placeholder images with actual photos
4. Set up backend for forms
5. Configure payment processing
6. Add SSL certificate
7. Launch!

---

**Remember**: All files must be in the same folder for the links to work properly!

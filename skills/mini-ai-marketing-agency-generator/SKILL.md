---
name: mini-ai-marketing-agency-generator
description: Generate complete marketing packages for small businesses using rule-based templates in vanilla JavaScript
triggers:
  - "generate marketing content for a business"
  - "create social media posts and ad copy"
  - "build a marketing brief package"
  - "generate brand positioning and content pillars"
  - "create bilingual marketing content"
  - "make rule-based marketing templates"
  - "generate business marketing materials"
  - "create marketing agency starter package"
---

# Mini AI Marketing Agency Generator

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## What It Does

Mini AI Marketing Agency is a browser-based marketing content generator that creates complete starter marketing packages for small businesses. It uses rule-based templates (no AI API calls) to generate:

- Brand positioning summaries
- 3 content pillars
- 5 post ideas
- 3 ready-to-use captions
- 2 ad copy options
- 1 static creative headline

Supports **bilingual output** (English/Burmese), **4 tones** (Friendly/Professional/Premium/Fun), and **2 platforms** (Facebook/Instagram).

## Installation

No build step required. Just clone and open:

```bash
git clone https://github.com/rover-aungkhine/mini-ai-marketing-agency.git
cd mini-ai-marketing-agency

# Option 1: Open directly in browser
open index.html

# Option 2: Serve locally
python3 -m http.server 8000
# Visit http://localhost:8000
```

## Project Structure

```
mini-ai-marketing-agency/
├── index.html    # Form + results UI
├── style.css     # Responsive styles with CSS custom properties
└── script.js     # Rule-based generation engine
```

## How the Engine Works

The generation engine is entirely client-side vanilla JavaScript:

1. **Form Validation** — validates 8 required fields
2. **Template Selection** — picks templates from banks keyed by `[language][tone][platform]`
3. **Content Mapping** — matches business type to relevant content pillars
4. **Interpolation** — replaces placeholders like `{name}`, `{product}`, `{audience}`
5. **DOM Rendering** — displays results in 6 sections with copy buttons

### Core Form Fields

```javascript
const formFields = {
  businessName: 'text',           // Business name
  businessType: 'text',           // e.g. "café", "boutique", "clinic"
  product: 'text',                // Product or service
  audience: 'text',               // Target audience
  offer: 'text',                  // Main offer
  platform: 'select',             // Facebook or Instagram
  tone: 'select',                 // Friendly/Professional/Premium/Fun
  language: 'select'              // English (en) or Burmese (my)
};
```

## Template Bank Structure

Templates are organized by language, tone, and platform:

```javascript
const templates = {
  positioning: {
    en: {
      friendly: [
        "We're {name} — bringing {product} to {audience} who want {offer}.",
        "{name} helps {audience} enjoy {product} without the hassle."
      ],
      professional: [
        "{name} delivers {product} designed for {audience}.",
        "Trusted by {audience}, {name} offers {offer}."
      ],
      premium: [
        "{name} — where {audience} experience {product} redefined.",
        "Exclusive {product} crafted for discerning {audience}."
      ],
      fun: [
        "{name} is your go-to for {product} that rocks!",
        "Hey {audience}! Ready for {product} that's actually fun?"
      ]
    },
    my: {
      friendly: [
        "{name} က {audience} အတွက် {product} ကို ပေးဆောင်ပါတယ်။",
        // ... Burmese templates
      ]
      // ... other tones
    }
  },
  adCopy: {
    facebook: {
      en: {
        friendly: [
          "🎉 {offer} — just for you! Visit {name} today.",
          "Your {product} journey starts here. {offer} available now!"
        ]
        // ... other tones
      }
    },
    instagram: {
      en: {
        friendly: [
          "✨ {offer} ✨\nTap to discover {product} at {name}",
          "{audience} — this one's for you! 💫 {offer}"
        ]
        // ... other tones
      }
    }
  }
};
```

## Content Pillar Mapping

Business type keywords map to relevant content themes:

```javascript
const contentPillarMap = {
  café: ['Behind the Beans', 'Customer Stories', 'Seasonal Specials'],
  coffee: ['Behind the Beans', 'Customer Stories', 'Seasonal Specials'],
  restaurant: ['Chef's Table', 'Customer Reviews', 'Daily Specials'],
  boutique: ['Style Tips', 'New Arrivals', 'Customer Spotlights'],
  fashion: ['Trend Alerts', 'Styling Guides', 'Behind the Brand'],
  clinic: ['Health Tips', 'Patient Success Stories', 'Service Highlights'],
  salon: ['Style Transformations', 'Beauty Tips', 'Client Features'],
  gym: ['Workout Tips', 'Member Wins', 'Fitness Challenges'],
  fitness: ['Exercise How-Tos', 'Transformation Stories', 'Nutrition Tips'],
  default: ['Our Story', 'Customer Wins', 'What's New']
};
```

## Real Usage Example

### Generating Marketing Content

```javascript
// This happens when user submits the form
function generateMarketing() {
  // 1. Validate form
  const errors = validateForm();
  if (errors.length > 0) {
    showErrors(errors);
    return;
  }

  // 2. Build context from form
  const context = {
    name: document.getElementById('businessName').value.trim(),
    type: document.getElementById('businessType').value.trim().toLowerCase(),
    product: document.getElementById('product').value.trim(),
    audience: document.getElementById('audience').value.trim(),
    offer: document.getElementById('offer').value.trim(),
    platform: document.getElementById('platform').value,
    tone: document.getElementById('tone').value,
    language: document.getElementById('language').value
  };

  // 3. Generate each section
  const positioning = generatePositioning(context);
  const pillars = generatePillars(context);
  const posts = generatePosts(context);
  const captions = generateCaptions(context);
  const ads = generateAds(context);
  const creative = generateCreative(context);

  // 4. Render results
  renderResults({
    positioning,
    pillars,
    posts,
    captions,
    ads,
    creative
  });

  // 5. Show results section
  document.getElementById('results').classList.remove('hidden');
  document.getElementById('results').scrollIntoView({ behavior: 'smooth' });
}
```

### Template Interpolation

```javascript
function interpolate(template, context) {
  return template
    .replace(/{name}/g, context.name)
    .replace(/{type}/g, context.type)
    .replace(/{product}/g, context.product)
    .replace(/{audience}/g, context.audience)
    .replace(/{offer}/g, context.offer);
}

// Example usage
const template = "We're {name} — bringing {product} to {audience}.";
const context = {
  name: "Sunrise Café",
  product: "artisan coffee",
  audience: "busy professionals"
};
const result = interpolate(template, context);
// "We're Sunrise Café — bringing artisan coffee to busy professionals."
```

### Generating Content Pillars

```javascript
function generatePillars(context) {
  const type = context.type.toLowerCase();
  
  // Find matching business type
  for (const [keyword, pillars] of Object.entries(contentPillarMap)) {
    if (type.includes(keyword)) {
      return pillars;
    }
  }
  
  // Return default if no match
  return contentPillarMap.default;
}
```

### Copy to Clipboard

```javascript
function copyToClipboard(text, buttonElement) {
  navigator.clipboard.writeText(text).then(() => {
    showToast('Copied to clipboard!');
    
    // Visual feedback
    const originalText = buttonElement.textContent;
    buttonElement.textContent = '✓ Copied';
    setTimeout(() => {
      buttonElement.textContent = originalText;
    }, 2000);
  }).catch(err => {
    console.error('Copy failed:', err);
    showToast('Copy failed. Please try again.', 'error');
  });
}
```

## Adding New Templates

To extend the template banks:

```javascript
// Add new tone
templates.positioning.en.casual = [
  "Yo! {name} here with {product} for {audience}.",
  "What's up {audience}? Check out {product} at {name}!"
];

// Add new language
templates.positioning.th = {
  friendly: [
    "{name} นำเสนอ {product} สำหรับ {audience}",
    // ... Thai templates
  ]
};

// Add new platform
templates.adCopy.tiktok = {
  en: {
    fun: [
      "🎵 {offer} alert! 🎵\n#{name} #{product}",
      "POV: You found the best {product} 😍 {offer}"
    ]
  }
};
```

## Adding New Business Types

```javascript
contentPillarMap.bakery = [
  'Daily Fresh Bakes',
  'Baker's Secrets',
  'Customer Favorites'
];

contentPillarMap.bookstore = [
  'Book Recommendations',
  'Author Spotlights',
  'Reader Reviews'
];
```

## Form Validation Pattern

```javascript
function validateForm() {
  const errors = [];
  const requiredFields = [
    { id: 'businessName', label: 'Business Name' },
    { id: 'businessType', label: 'Business Type' },
    { id: 'product', label: 'Product/Service' },
    { id: 'audience', label: 'Target Audience' },
    { id: 'offer', label: 'Main Offer' }
  ];

  requiredFields.forEach(field => {
    const value = document.getElementById(field.id).value.trim();
    if (!value) {
      errors.push(`${field.label} is required`);
    }
  });

  return errors;
}
```

## Responsive Design

Uses CSS custom properties for theming:

```css
:root {
  --primary: #6366f1;
  --primary-dark: #4f46e5;
  --bg: #f9fafb;
  --card-bg: #ffffff;
  --text: #1f2937;
  --border: #e5e7eb;
  --success: #10b981;
  --radius: 12px;
  --shadow: 0 1px 3px rgba(0,0,0,0.1);
}

/* Mobile-first responsive cards */
.output-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1.5rem;
}

@media (min-width: 768px) {
  .output-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
```

## Bilingual Font Support

```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Sans+Myanmar:wght@400;500;600;700&display=swap" rel="stylesheet">
```

```css
body {
  font-family: 'Inter', 'Noto Sans Myanmar', sans-serif;
}
```

## Common Patterns

### Reset Form

```javascript
function resetForm() {
  document.getElementById('briefForm').reset();
  document.getElementById('results').classList.add('hidden');
  showToast('Form reset');
}
```

### Copy All Results

```javascript
function copyAllResults() {
  const sections = ['positioning', 'pillars', 'posts', 'captions', 'ads', 'creative'];
  const allText = sections.map(id => {
    const element = document.getElementById(id);
    return element ? element.textContent : '';
  }).join('\n\n---\n\n');
  
  copyToClipboard(allText, event.target);
}
```

## Troubleshooting

**Templates not showing correctly:**
- Check language/tone/platform combination exists in template bank
- Verify interpolation placeholders match context keys exactly

**Copy button not working:**
- Ensure page is served over HTTPS or localhost (clipboard API requirement)
- Check browser clipboard permissions

**Burmese text rendering incorrectly:**
- Verify Noto Sans Myanmar font is loaded
- Check `lang` attribute is set correctly

**Content pillars always showing default:**
- Business type keyword matching is case-insensitive and uses `.includes()`
- Add more keywords to `contentPillarMap` for better matching

## Extending the Project

### Add New Output Section

1. Add HTML card to `index.html`:
```html
<div class="output-card">
  <h3>Email Subject Lines</h3>
  <div id="emails" class="output-content"></div>
  <button onclick="copySection('emails')">Copy</button>
</div>
```

2. Create template bank in `script.js`:
```javascript
templates.emails = {
  en: {
    friendly: [
      "{offer} just for you! 💌",
      "Hey {audience}, check this out..."
    ]
  }
};
```

3. Add generation function:
```javascript
function generateEmails(context) {
  const template = selectRandom(
    templates.emails[context.language][context.tone]
  );
  return interpolate(template, context);
}
```

This skill enables AI coding agents to understand, modify, and extend the Mini AI Marketing Agency template generator effectively.

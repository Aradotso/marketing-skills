---
name: email-marketing-regulations-compliance
description: Navigate global email marketing laws (GDPR, CAN-SPAM, CASL, LGPD) with comprehensive regulatory guidance from EmailOctopus
triggers:
  - check email marketing laws for a country
  - what are the GDPR requirements for email marketing
  - do I need consent for email marketing in Canada
  - CAN-SPAM compliance requirements
  - email marketing regulations for Brazil
  - what penalties exist for spam violations
  - soft opt-in rules for email marketing
  - verify email marketing compliance
---

# Email Marketing Regulations Compliance

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides access to the comprehensive email marketing regulations repository maintained by EmailOctopus. It contains detailed information about email marketing legislation across major jurisdictions including GDPR (Europe), CAN-SPAM (USA), CASL (Canada), LGPD (Brazil), and many others.

## What This Resource Provides

The email-marketing-regulations repository is a reference guide that documents:

- **Legal requirements** for email marketing in 20+ countries
- **Consent requirements** (opt-in, opt-out, soft opt-in rules)
- **Content requirements** (sender identification, unsubscribe mechanisms)
- **Penalty structures** for non-compliance
- **Key terminology** (explicit vs implied consent, double opt-in)
- **Regional variations** in data protection laws

This is a documentation resource, not a software library or CLI tool. It's designed for research and compliance planning.

## Installation

Clone the repository to access the documentation locally:

```bash
git clone https://github.com/threeheartsdigital/email-marketing-regulations.git
cd email-marketing-regulations
```

Alternatively, browse the documentation directly on GitHub or reference specific country files as needed.

## Repository Structure

```
email-marketing-regulations/
├── README.md                    # Overview table of all countries
├── country/
│   ├── australia.md
│   ├── belgium.md
│   ├── brazil.md
│   ├── canada.md
│   ├── china.md
│   ├── denmark.md
│   ├── finland.md
│   ├── germany.md
│   ├── hongkong.md
│   ├── iceland.md
│   ├── india.md
│   ├── ireland.md
│   ├── israel.md
│   ├── japan.md
│   ├── singapore.md
│   ├── south-africa.md
│   ├── uae.md
│   ├── uk.md
│   └── usa.md
```

## Key Regulatory Frameworks

### GDPR (European Union + UK)

**Consent Requirements:**
- Prior explicit consent required for marketing emails
- Soft opt-in exception: existing customers, similar products/services
- Must offer easy opt-out at collection and in every message

**Content Requirements:**
- Clear sender identification
- Valid contact address
- Transparent processing information

**Penalties:**
- Up to €20 million or 4% of annual global turnover (whichever is higher)
- UK: Up to £17.5 million or 4% of annual global turnover

**Example Compliance Checklist:**
```markdown
- [ ] Obtained explicit consent via unticked checkbox
- [ ] Provided clear information about data processing
- [ ] Included unsubscribe link in email footer
- [ ] Sender name and physical address visible
- [ ] Consent records stored with timestamp and method
- [ ] Privacy policy accessible and up-to-date
```

### CAN-SPAM (United States)

**Consent Requirements:**
- No prior consent required (opt-out model)
- Must honor opt-out requests within 10 business days

**Content Requirements:**
- Accurate "From" and "To" information
- Non-deceptive subject lines
- Physical postal address of sender
- Clear identification as advertisement
- Conspicuous opt-out mechanism

**Penalties:**
- Up to $53,088 USD per violation
- Additional penalties for aggravated violations

**Example Compliant Email Footer:**
```html
<footer>
  <p><strong>Acme Corporation</strong><br>
  123 Main Street, Suite 100<br>
  San Francisco, CA 94102<br>
  United States</p>
  
  <p>This is a promotional email. If you no longer wish to receive these messages, 
  <a href="https://example.com/unsubscribe?email={{email}}">click here to unsubscribe</a>.</p>
  
  <p>Questions? Contact us at support@example.com</p>
</footer>
```

### CASL (Canada)

**Consent Requirements:**
- Express consent (written or oral) required by default
- Implied consent for existing business relationships (up to 24 months after transaction)
- Soft opt-in during sales inquiry (6 months)

**Content Requirements:**
- Sender identification (name and contact info)
- Functional unsubscribe mechanism
- Physical mailing address or other contact information

**Penalties:**
- Up to $10 million CAD per violation

**Example Express Consent Form:**
```html
<form action="/subscribe" method="post">
  <label for="email">Email Address:</label>
  <input type="email" id="email" name="email" required>
  
  <label>
    <input type="checkbox" name="consent" value="yes" required>
    I consent to receive commercial electronic messages from Acme Corp 
    about products, services, and special offers. I understand I can 
    unsubscribe at any time.
  </label>
  
  <p><small>We respect your privacy. See our 
  <a href="/privacy">Privacy Policy</a> for details on how we handle your data.</small></p>
  
  <button type="submit">Subscribe</button>
</form>
```

### LGPD (Brazil)

**Consent Requirements:**
- Consent or documented legitimate interest required
- No statutory soft opt-in provision
- Clear, specific consent statements

**Content Requirements:**
- Transparent sender identification
- Easy opt-out mechanism
- Clear purpose of data processing

**Penalties:**
- Up to 2% of revenue from Brazil
- Maximum 50 million BRL per infraction

**Example Consent Record Structure:**
```json
{
  "subscriber_id": "sub_abc123",
  "email": "user@example.com.br",
  "consent_timestamp": "2026-08-15T14:30:00Z",
  "consent_method": "website_signup",
  "consent_ip": "203.0.113.42",
  "consent_text": "Eu concordo em receber e-mails de marketing da Acme Corp sobre produtos e serviços.",
  "legal_basis": "consent",
  "purpose": "marketing_communications",
  "data_controller": "Acme Corp LTDA",
  "opt_out_available": true,
  "opt_out_url": "https://example.com.br/cancelar-inscricao"
}
```

## Common Compliance Patterns

### Double Opt-In Implementation

Most stringent jurisdictions benefit from double opt-in:

```javascript
// Example Node.js/Express signup flow
const crypto = require('crypto');

app.post('/api/subscribe', async (req, res) => {
  const { email, consentGiven, ipAddress, userAgent } = req.body;
  
  if (!consentGiven) {
    return res.status(400).json({ error: 'Consent required' });
  }
  
  // Generate confirmation token
  const confirmToken = crypto.randomBytes(32).toString('hex');
  
  // Store pending subscription
  await db.pendingSubscribers.create({
    email,
    confirmToken,
    consentTimestamp: new Date(),
    ipAddress,
    userAgent,
    status: 'pending_confirmation'
  });
  
  // Send confirmation email
  await sendEmail({
    to: email,
    subject: 'Please confirm your subscription',
    html: `
      <p>Click the link below to confirm your subscription:</p>
      <a href="${process.env.BASE_URL}/confirm/${confirmToken}">Confirm Subscription</a>
      <p>If you didn't request this, you can safely ignore this email.</p>
    `
  });
  
  res.json({ success: true, message: 'Confirmation email sent' });
});

app.get('/confirm/:token', async (req, res) => {
  const pending = await db.pendingSubscribers.findOne({
    confirmToken: req.params.token
  });
  
  if (!pending) {
    return res.status(404).send('Invalid or expired confirmation link');
  }
  
  // Move to confirmed subscribers
  await db.subscribers.create({
    email: pending.email,
    confirmedAt: new Date(),
    consentMethod: 'double_opt_in',
    sourceIp: pending.ipAddress,
    originalConsentTimestamp: pending.consentTimestamp
  });
  
  await db.pendingSubscribers.delete({ confirmToken: req.params.token });
  
  res.send('Subscription confirmed! Thank you.');
});
```

### Soft Opt-In Qualification Check

```python
# Example Python function to determine if soft opt-in applies
from datetime import datetime, timedelta
from enum import Enum

class ConsentType(Enum):
    EXPLICIT = "explicit"
    SOFT_OPT_IN = "soft_opt_in"
    NONE = "none"

def check_soft_optin_eligibility(
    customer_email: str,
    last_purchase_date: datetime,
    products_purchased: list[str],
    marketing_products: list[str],
    gave_optin_at_purchase: bool,
    country: str
) -> ConsentType:
    """
    Determine if soft opt-in applies based on country regulations.
    
    Note: This is simplified logic. Always consult with legal counsel.
    """
    
    # Soft opt-in generally requires:
    # 1. Contact details obtained during a sale
    # 2. Marketing for similar products/services
    # 3. Easy opt-out provided at collection
    # 4. Within time limits (varies by country)
    
    time_limits = {
        "GB": timedelta(days=365),  # UK: reasonable period
        "DE": timedelta(days=365),  # Germany: similar products
        "IE": timedelta(days=365),  # Ireland: 12 months per regulation
        "AU": timedelta(days=730),  # Australia: reasonable period
        "BE": timedelta(days=365),  # Belgium: similar to GDPR
    }
    
    if country not in time_limits:
        return ConsentType.NONE
    
    # Check time limit
    time_since_purchase = datetime.now() - last_purchase_date
    if time_since_purchase > time_limits[country]:
        return ConsentType.NONE
    
    # Check if products are similar
    if not any(p in marketing_products for p in products_purchased):
        return ConsentType.NONE
    
    # Check if opt-out was offered at purchase
    if not gave_optin_at_purchase:
        return ConsentType.NONE
    
    return ConsentType.SOFT_OPT_IN

# Usage example
eligibility = check_soft_optin_eligibility(
    customer_email="customer@example.com",
    last_purchase_date=datetime(2026, 6, 1),
    products_purchased=["web_hosting", "domain_registration"],
    marketing_products=["ssl_certificates", "email_hosting"],
    gave_optin_at_purchase=True,
    country="GB"
)

if eligibility == ConsentType.SOFT_OPT_IN:
    print("Soft opt-in applies - may send marketing for similar services")
elif eligibility == ConsentType.EXPLICIT:
    print("Explicit consent required")
else:
    print("No valid consent basis - do not send marketing")
```

### Unsubscribe Management

```ruby
# Example Ruby/Rails unsubscribe handler
class UnsubscribeController < ApplicationController
  skip_before_action :verify_authenticity_token, only: [:one_click]
  
  # One-click unsubscribe (RFC 8058 - recommended for compliance)
  def one_click
    subscriber = Subscriber.find_by(unsubscribe_token: params[:token])
    
    if subscriber
      subscriber.update!(
        status: 'unsubscribed',
        unsubscribed_at: Time.current,
        unsubscribe_method: 'one_click',
        unsubscribe_ip: request.remote_ip
      )
      
      # Log for compliance records
      AuditLog.create!(
        event: 'unsubscribe',
        subscriber_id: subscriber.id,
        timestamp: Time.current,
        details: { method: 'one_click', ip: request.remote_ip }
      )
      
      head :ok
    else
      head :not_found
    end
  end
  
  # Web-based unsubscribe page
  def show
    @subscriber = Subscriber.find_by(unsubscribe_token: params[:token])
    
    unless @subscriber
      render :invalid_token and return
    end
  end
  
  def create
    subscriber = Subscriber.find_by(unsubscribe_token: params[:token])
    
    if subscriber
      # Honor unsubscribe within 10 business days (CAN-SPAM)
      subscriber.update!(
        status: 'unsubscribed',
        unsubscribed_at: Time.current,
        unsubscribe_method: 'web_form',
        unsubscribe_reason: params[:reason]
      )
      
      # Propagate to all systems immediately
      UnsubscribeJob.perform_async(subscriber.email)
      
      redirect_to unsubscribe_confirmed_path
    else
      render :invalid_token
    end
  end
end

# Email template with compliant unsubscribe
# app/views/mailers/newsletter.html.erb
# <footer>
#   <p>{{ company_name }}<br>
#   {{ physical_address }}</p>
#   
#   <p><a href="<%= unsubscribe_url(token: @subscriber.unsubscribe_token) %>">
#     Unsubscribe
#   </a> | <a href="<%= preferences_url(token: @subscriber.unsubscribe_token) %>">
#     Update Preferences
#   </a></p>
# </footer>
```

## Consent Record Keeping

Maintaining detailed consent records is critical for compliance:

```typescript
// TypeScript interface for comprehensive consent records
interface ConsentRecord {
  subscriberId: string;
  email: string;
  
  // Consent details
  consentTimestamp: Date;
  consentMethod: 'web_form' | 'api' | 'import' | 'soft_optin' | 'other';
  consentType: 'explicit' | 'implied' | 'soft_optin';
  consentVersion: string; // Privacy policy version
  
  // Technical details
  ipAddress: string;
  userAgent: string;
  referrerUrl?: string;
  
  // Legal basis
  legalBasis: 'consent' | 'legitimate_interest' | 'contract' | 'legal_obligation';
  jurisdiction: string; // ISO country code
  
  // Consent text shown to user
  consentLanguage: string; // ISO language code
  consentText: string;
  privacyPolicyUrl: string;
  
  // Scope
  purposes: string[]; // e.g., ['newsletter', 'product_updates', 'promotions']
  dataCategories: string[]; // e.g., ['email', 'name', 'preferences']
  
  // Lifecycle
  confirmed: boolean;
  confirmationTimestamp?: Date;
  revokedAt?: Date;
  revokeMethod?: string;
  
  // Audit trail
  lastUpdated: Date;
  updatedBy: string;
}

// Example database schema (PostgreSQL)
const createTableSQL = `
CREATE TABLE consent_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subscriber_id VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL,
  
  consent_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
  consent_method VARCHAR(50) NOT NULL,
  consent_type VARCHAR(50) NOT NULL,
  consent_version VARCHAR(50) NOT NULL,
  
  ip_address INET NOT NULL,
  user_agent TEXT,
  referrer_url TEXT,
  
  legal_basis VARCHAR(50) NOT NULL,
  jurisdiction VARCHAR(2) NOT NULL,
  
  consent_language VARCHAR(5) NOT NULL,
  consent_text TEXT NOT NULL,
  privacy_policy_url TEXT NOT NULL,
  
  purposes JSONB NOT NULL,
  data_categories JSONB NOT NULL,
  
  confirmed BOOLEAN DEFAULT FALSE,
  confirmation_timestamp TIMESTAMP WITH TIME ZONE,
  revoked_at TIMESTAMP WITH TIME ZONE,
  revoke_method VARCHAR(50),
  
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_by VARCHAR(255),
  
  INDEX idx_subscriber_id (subscriber_id),
  INDEX idx_email (email),
  INDEX idx_consent_timestamp (consent_timestamp),
  INDEX idx_jurisdiction (jurisdiction)
);
`;
```

## Multi-Jurisdiction Compliance Strategy

```yaml
# Example compliance configuration file
# compliance-config.yml

jurisdictions:
  EU:
    legislation: GDPR
    consent_required: explicit
    soft_optin_allowed: true
    soft_optin_conditions:
      - existing_customer
      - similar_products
      - opt_out_offered
    content_requirements:
      - sender_identification
      - contact_address
      - unsubscribe_link
    retention_limits:
      marketing_consent: 2_years
      consent_records: 6_years
    penalties:
      max_fine: "€20M or 4% global turnover"
  
  US:
    legislation: CAN-SPAM
    consent_required: false
    opt_out_model: true
    content_requirements:
      - accurate_header
      - truthful_subject
      - physical_address
      - clear_advertisement
      - conspicuous_unsubscribe
    opt_out_processing_days: 10
    penalties:
      per_violation: "$53,088 USD"
  
  CA:
    legislation: CASL
    consent_required: express
    implied_consent_duration:
      business_relationship: 24_months
      inquiry: 6_months
    content_requirements:
      - sender_identification
      - contact_info
      - unsubscribe_mechanism
    penalties:
      max_fine: "$10M CAD"
  
  BR:
    legislation: LGPD
    consent_required: true
    soft_optin_allowed: false
    legal_bases:
      - consent
      - legitimate_interest
    content_requirements:
      - transparent_sender
      - opt_out_mechanism
    penalties:
      max_fine: "2% revenue (Brazil), up to 50M BRL"

# Usage in application
def get_compliance_rules(country_code)
  jurisdiction = map_country_to_jurisdiction(country_code)
  COMPLIANCE_CONFIG['jurisdictions'][jurisdiction]
end
```

## Quick Reference Checklist

Use this checklist when setting up email marketing campaigns:

```markdown
## Pre-Campaign Compliance Checklist

### Consent & Legal Basis
- [ ] Identified target jurisdictions
- [ ] Determined applicable laws (GDPR, CAN-SPAM, CASL, etc.)
- [ ] Obtained appropriate consent type (explicit/implied/soft opt-in)
- [ ] Documented consent records with timestamps
- [ ] Verified soft opt-in eligibility if applicable

### Content Requirements
- [ ] Sender clearly identified (no misleading "From" names)
- [ ] Physical mailing address included in footer
- [ ] Subject line is truthful and not deceptive
- [ ] Email marked as advertisement if required by jurisdiction
- [ ] Privacy policy linked and accessible

### Unsubscribe Mechanism
- [ ] Unsubscribe link present and conspicuous
- [ ] Unsubscribe process free of charge
- [ ] Unsubscribe process requires minimal steps (one-click preferred)
- [ ] System processes unsubscribes within required timeframe
- [ ] Preference center available (optional but recommended)

### Data Protection
- [ ] Data minimization applied (collect only necessary data)
- [ ] Secure storage of personal data and consent records
- [ ] Data retention policies in place
- [ ] Data processing agreement with ESP if applicable
- [ ] Cross-border data transfer mechanisms compliant

### Testing & Monitoring
- [ ] Test email renders correctly
- [ ] All links functional (especially unsubscribe)
- [ ] Suppression list applied to remove unsubscribed users
- [ ] Bounce handling configured
- [ ] Complaint monitoring active
```

## Country-Specific Resources

To access detailed information for a specific country, reference the individual country files:

```bash
# View regulations for a specific country
cat country/canada.md
cat country/germany.md
cat country/australia.md

# Search for specific terms across all countries
grep -r "soft opt-in" country/
grep -r "double opt-in" country/

# Find penalty information
grep -r "Penalties" country/ -A 5
```

## Integration with Email Service Providers

Most ESPs provide compliance features. Here's how to verify your setup:

```javascript
// Example ESP compliance check (pseudo-code)
async function verifyESPCompliance(config) {
  const checks = {
    unsubscribeLink: false,
    physicalAddress: false,
    listHygiene: false,
    consentTracking: false,
    doubleOptIn: false
  };
  
  // Check template has required elements
  const template = await esp.getTemplate(config.templateId);
  
  if (template.content.includes('{{unsubscribe_url}}')) {
    checks.unsubscribeLink = true;
  }
  
  if (template.content.includes(config.physicalAddress)) {
    checks.physicalAddress = true;
  }
  
  // Check list settings
  const listSettings = await esp.getListSettings(config.listId);
  
  if (listSettings.doubleOptIn) {
    checks.doubleOptIn = true;
  }
  
  if (listSettings.automaticBounceHandling) {
    checks.listHygiene = true;
  }
  
  // Check consent tracking
  const subscriber = await esp.getSubscriber(config.listId, 'test@example.com');
  
  if (subscriber.consentTimestamp && subscriber.consentMethod) {
    checks.consentTracking = true;
  }
  
  return checks;
}

// Environment-specific compliance
const ESP_API_KEY = process.env.ESP_API_KEY;
const COMPLIANCE_MODE = process.env.COMPLIANCE_MODE; // 'GDPR', 'CASL', 'CAN_SPAM'
```

## Troubleshooting Common Compliance Issues

### Issue: Determining Which Laws Apply

**Problem:** Unsure which regulations govern your email marketing.

**Solution:**
1. Identify where you are located (sender jurisdiction)
2. Identify where your recipients are located (recipient jurisdiction)
3. Identify where email is processed (server location)
4. Apply the most stringent applicable law

Reference the README.md table to see requirements at a glance, then review specific country files.

### Issue: Soft Opt-In vs. Explicit Consent

**Problem:** Confusion about when soft opt-in can be used.

**Solution:**
- Soft opt-in is an **exception**, not a default
- Only applies when ALL conditions are met:
  - Contact details obtained during a sale/inquiry
  - Marketing is for YOUR OWN similar products
  - Opt-out was offered when collecting details
  - Opt-out is in every message
  - Within time limits (varies by country)
- When in doubt, use explicit consent (safer)

### Issue: International Audience

**Problem:** Subscribers across multiple jurisdictions.

**Solution:**
```python
# Segment lists by jurisdiction and apply appropriate rules
def segment_by_compliance_region(subscribers):
    regions = {
        'GDPR': [],      # EU + EEA
        'CASL': [],      # Canada
        'CAN_SPAM': [],  # United States
        'LGPD': [],      # Brazil
        'OTHER': []
    }
    
    for subscriber in subscribers:
        country = subscriber.country_code
        
        if country in ['AT', 'BE', 'BG', 'HR', 'CY', 'CZ', 'DK', 'EE', 
                       'FI', 'FR', 'DE', 'GR', 'HU', 'IE', 'IT', 'LV', 
                       'LT', 'LU', 'MT', 'NL', 'PL', 'PT', 'RO', 'SK', 
                       'SI', 'ES', 'SE', 'IS', 'LI', 'NO']:
            regions['GDPR'].append(subscriber)
        elif country == 'CA':
            regions['CASL'].append(subscriber)
        elif country == 'US':
            regions['CAN_SPAM'].append(subscriber)
        elif country == 'BR':
            regions['LGPD'].append(subscriber)
        else:
            regions['OTHER'].append(subscriber)
    
    return regions

# Apply strictest standard globally for simplicity
# (Many companies default to GDPR compliance worldwide)
```

### Issue: Consent Records Missing

**Problem:** Historic subscribers without documented consent.

**Solution:**
1. **Do not** retroactively claim consent
2. Send re-permission campaign to reconfirm interest
3. Remove non-responders after reasonable period
4. Implement proper consent tracking going forward

```html
<!-- Re-permission email template -->
<h2>We Value Your Privacy</h2>

<p>We're updating our systems to ensure we have your explicit permission 
to send you marketing emails. We'd love to stay in touch!</p>

<p>Please confirm you'd like to continue receiving our newsletter:</p>

<a href="{{reconfirm_url}}" style="background: #007bff; color: white; 
padding: 10px 20px; text-decoration: none; display: inline-block;">
  Yes, Keep Me Subscribed
</a>

<p><small>If you don't confirm by {{deadline_date}}, you'll be 
automatically unsubscribed. You can always resubscribe later 
at {{signup_url}}.</small></p>
```

## Legal Disclaimer

This skill provides general information about email marketing regulations based on the open-source repository maintained by EmailOctopus and contributors. It is **not legal advice**. 

- Laws change frequently
- Information may be incomplete or outdated
- Interpretation varies by jurisdiction
- Every situation is unique

**Always consult with a qualified attorney** in the relevant jurisdiction before conducting email marketing campaigns. The repository maintainers, contributors, and this skill author accept no liability for decisions made based on this information.

## Additional Resources

- **Repository**: https://github.com/threeheartsdigital/email-marketing-regulations
- **EmailOctopus**: https://emailoctopus.com
- **GDPR Official Text**: https://gdpr.eu
- **CAN-SPAM Act**: https://www.ftc.gov/tips-advice/business-center/guidance/can-spam-act-compliance-guide-business
- **CASL**: https://crtc.gc.ca/eng/internet/anti.htm
- **LGPD**: https://www.gov.br/cidadania/pt-br/acesso-a-informacao/lgpd

---

**Last Updated**: 2026-07-26 (based on repository metadata)

**Skill Version**: 1.0.0

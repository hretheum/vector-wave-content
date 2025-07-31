# Make.com Automation Setup dla AI Agents Content

## 🎯 Scenario 1: "AI Agent Content Processor"

### Trigger: Watch Folder
```
Module: Google Drive - Watch Files in a Folder
Folder: /growth automation/pure content/content/2025-01-27-ai-agents/
Schedule: Every 15 minutes
```

### Step 1: Read Content Files
```
Module: Google Drive - Download a File
Map: File ID from trigger
```

### Step 2: Parse Content by Platform
```
Module: Text Parser - Match Pattern
Pattern: 
  - If filename contains "twitter" → Extract Twitter sections
  - If filename contains "linkedin" → Extract LinkedIn sections  
  - If filename contains "beehiiv" → Extract Newsletter content
```

### Step 3: AI Agent Enhancement (OPTIONAL)
```
Module: OpenAI - Create Completion
Prompt: "Enhance this {platform} post about AI agents. 
Keep core message but add engagement hooks.
Original: {content}"
Max tokens: 500
```

### Step 4: Store in Airtable
```
Module: Airtable - Create Record
Table: Content_Pipeline
Fields:
  - Title: {extracted_title}
  - Platform: {platform}
  - Content: {parsed_content}
  - Status: "Scheduled"
  - Publish_Time: {scheduled_time}
  - Campaign: "AI_Agents_Launch"
```

### Step 5: Create Publishing Tasks
```
Module: Router
Route 1: If platform = "Twitter" AND time = "18:00 Sunday"
  → Create Twitter Draft
Route 2: If platform = "LinkedIn" AND time = "09:00 Monday"  
  → Create LinkedIn Draft
Route 3: If platform = "Beehiiv" AND time = "09:00 Tuesday"
  → Create Newsletter Draft
```

---

## 🚀 Scenario 2: "Multi-Platform Publisher"

### Trigger: Schedule
```
Module: Schedule
Setting: 
  - Sunday 18:00 (Twitter teaser)
  - Monday 08:00 (Twitter thread)
  - Monday 09:00 (LinkedIn)
  - Monday 14:00 (Twitter follow-up)
  - Tuesday 09:00 (Beehiiv)
```

### Step 1: Get Content from Airtable
```
Module: Airtable - Search Records
Table: Content_Pipeline
Formula: AND(
  {Status} = "Scheduled",
  {Publish_Time} <= NOW(),
  {Campaign} = "AI_Agents_Launch"
)
```

### Step 2: Platform Router
```
Module: Router
Routes based on {Platform} field
```

### Route A: Twitter Publishing
```
Module: HTTP - Make a Request
URL: https://api.twitter.com/2/tweets
Headers: Authorization Bearer {TWITTER_API_KEY}
Body: {
  "text": {content},
  "reply_settings": "everyone"
}
```

### Route B: LinkedIn Publishing  
```
Module: HTTP - Make a Request
URL: https://api.linkedin.com/v2/ugcPosts
Headers: Authorization Bearer {LINKEDIN_TOKEN}
Body: {
  "author": "urn:li:person:{YOUR_ID}",
  "lifecycleState": "PUBLISHED",
  "specificContent": {
    "com.linkedin.ugc.ShareContent": {
      "shareCommentary": {
        "text": {content}
      }
    }
  }
}
```

### Route C: Beehiiv Publishing
```
Module: HTTP - Make a Request
URL: https://api.beehiiv.com/v1/publications/{PUB_ID}/posts
Headers: Authorization Bearer {BEEHIIV_API_KEY}
Body: {
  "content": {content},
  "status": "draft",
  "send_at": {scheduled_time}
}
```

### Step 3: Update Status
```
Module: Airtable - Update Record
Record ID: {record_id}
Fields:
  - Status: "Published"
  - Published_At: NOW()
  - Platform_ID: {response.id}
```

### Step 4: Log Analytics
```
Module: Google Sheets - Add Row
Spreadsheet: Analytics_Tracking
Values:
  - Timestamp: NOW()
  - Platform: {platform}
  - Content_ID: {platform_id}
  - Campaign: "AI_Agents_Launch"
```

---

## 📊 Scenario 3: "Engagement Tracker & Amplifier"

### Trigger: Every 30 minutes
```
Module: Schedule
Interval: 30 minutes
Days: Monday-Friday
```

### Step 1: Check Platform Metrics
```
Module: Router
3 parallel routes for each platform
```

### Twitter Metrics Route
```
Module: HTTP - Twitter API
Endpoint: /2/tweets/{tweet_id}?tweet.fields=public_metrics
Store: likes, retweets, replies, impressions
```

### LinkedIn Metrics Route
```
Module: HTTP - LinkedIn API  
Endpoint: /v2/socialActions/{post_id}
Store: reactions, comments, shares
```

### Beehiiv Metrics Route
```
Module: HTTP - Beehiiv API
Endpoint: /v1/publications/{pub_id}/posts/{post_id}/stats
Store: opens, clicks, forwards
```

### Step 2: Aggregate Performance
```
Module: Airtable - Update Record
Find record by Platform_ID
Update fields:
  - Impressions: {metrics.impressions}
  - Engagement: {metrics.engagements}
  - CTR: {metrics.clicks/metrics.impressions}
  - Status: IF(engagement > 100, "Viral", "Normal")
```

### Step 3: Viral Detection
```
Module: Filter
Condition: Engagement > 100 OR Growth_Rate > 200%
```

### Step 4: Create Follow-up Content
```
Module: OpenAI - Create Completion
Prompt: "Create a follow-up {platform} post for this viral content:
Original: {content}
Metrics: {engagement_data}
Generate excitement-building follow-up"
```

### Step 5: Notify & Schedule Boost
```
Module: Slack - Send Message
Channel: #content-alerts
Message: "🔥 VIRAL ALERT: {platform} post at {engagement} engagements!"

Module: Airtable - Create Record
Table: Content_Pipeline
Fields:
  - Content: {follow_up_content}
  - Status: "Scheduled"
  - Publish_Time: NOW() + 2 hours
  - Type: "Viral_Followup"
```

---

## 🛠️ Scenario 4: "Lead Capture from Content"

### Trigger: Webhook
```
Module: Webhooks - Custom Webhook
Listen for: Mentions, DMs, Comments containing keywords:
- "how to implement"
- "consultation"
- "more info"
- "github"
```

### Step 1: Parse Incoming Data
```
Module: JSON - Parse JSON
Extract: platform, user_handle, message, user_url
```

### Step 2: Categorize Intent
```
Module: OpenAI - Create Completion
Prompt: "Categorize this message intent:
Message: {message}
Categories: [Info_Request, Business_Inquiry, Technical_Question, Other]"
```

### Step 3: Auto-Response Router
```
Module: Router
Based on {intent_category}
```

### Route: Business Inquiry
```
Module: HTTP - Send DM/Reply
Message: "Thanks for your interest! I'd love to discuss how AI agents can transform your development workflow. 

Book a free consultation: calendly.com/hretheum/ai-agents

Or check out the open-source code: github.com/hretheum/detektr"
```

### Route: Technical Question
```
Module: HTTP - Send Reply
Message: "Great question! Here's a quick answer: {brief_answer}

For detailed implementation, check our repo: github.com/hretheum/detektr

The README has setup instructions and examples."
```

### Step 4: Add to CRM
```
Module: Airtable - Create Record
Table: Leads
Fields:
  - Name: {user_handle}
  - Platform: {platform}
  - Intent: {intent_category}
  - Message: {original_message}
  - Source: "AI_Agents_Campaign"
  - Date: NOW()
```

---

## 🔐 Configuration Variables

```javascript
// Store in Make.com Data Store
{
  "api_keys": {
    "twitter": "YOUR_TWITTER_API_KEY",
    "linkedin": "YOUR_LINKEDIN_TOKEN",
    "beehiiv": "YOUR_BEEHIIV_API_KEY",
    "openai": "YOUR_OPENAI_KEY"
  },
  "content_settings": {
    "viral_threshold": 100,
    "followup_delay_hours": 2,
    "max_posts_per_day": 5
  },
  "platforms": {
    "twitter": {
      "optimal_times": ["08:00", "14:00", "18:00"],
      "hashtags": ["#AIAgents", "#Automation", "#OpenSource"]
    },
    "linkedin": {
      "optimal_times": ["09:00", "12:00", "17:00"],
      "max_length": 3000
    }
  }
}
```

---

## 🚦 Testing Checklist

1. **Dry Run Mode**
   - Set all HTTP modules to "Test" mode
   - Use Airtable sandbox base
   - Log outputs instead of posting

2. **Content Validation**
   - Check character limits per platform
   - Verify image attachments
   - Test link previews

3. **Error Handling**
   - Add error handlers to each HTTP module
   - Set up fallback notifications
   - Create retry logic for API failures

4. **Monitoring**
   - Set up Make.com notifications
   - Create daily summary scenario
   - Track API usage limits

---

## 💡 Pro Tips

1. **Use Make.com AI Agents** for content enhancement
2. **Store templates** in Data Store for reuse
3. **Version control** your scenarios (export regularly)
4. **A/B test** posting times using Router probability
5. **Rate limit** aware - add delays between API calls

Ready to automate! 🚀
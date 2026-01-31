# Ultimate Test 06: Multi-Platform Content Publishing Skill

**Trigger:** "publish [CONTENT] to [PLATFORMS]" or "post about [TOPIC] everywhere"

**Expected Duration:** 5-10 minutes for full multi-platform publish

---

## Skill Instructions

When user asks to publish content, follow these steps exactly:

### Phase 1: Content Preparation

1. **Parse request**: Identify:
   - Content source: direct text, file path, or URL
   - Target platforms: list of platforms (e.g., "twitter, linkedin, blog")
   - Schedule: immediate or specific time
   - IF platforms not specified: Ask user or default to all configured

2. **Create publishing workspace**:
   - Create `OUTPUT/publish_[timestamp]/`
   - Write `campaign.json`:
     ```json
     {
       "campaign_id": "[timestamp]",
       "original_content": "[source]",
       "platforms": ["platform1", "platform2"],
       "created_at": "[timestamp]",
       "status": "preparing"
     }
     ```

3. **Load source content**:
   - IF file path: `file_read` and extract content
   - IF URL: `web_fetch` and extract main content
   - IF direct text: Use as-is
   - Save original to `OUTPUT/publish_[timestamp]/original.md`

4. **Analyze content**:
   - Word count
   - Key topics/themes
   - Tone (professional, casual, promotional)
   - Media references (images, links)
   - Write analysis to `OUTPUT/publish_[timestamp]/analysis.md`

### Phase 2: Platform Adaptation

5. **FOR EACH target platform**, create adapted version:

   **Twitter/X:**
   a. IF content > 280 chars: Create thread (split at sentence boundaries)
   b. Extract hashtags from key topics (max 3)
   c. Shorten URLs if present
   d. Write to `OUTPUT/publish_[timestamp]/twitter.md`

   **LinkedIn:**
   a. Professional tone adjustment
   b. Add hook in first line (appears in preview)
   c. Include call-to-action
   d. Format with line breaks for readability
   e. Max 3000 characters
   f. Write to `OUTPUT/publish_[timestamp]/linkedin.md`

   **Blog/Website:**
   a. Full content with formatting
   b. Add SEO title and meta description
   c. Suggest featured image description
   d. Add internal/external links if relevant
   e. Write to `OUTPUT/publish_[timestamp]/blog.md`

   **Email Newsletter:**
   a. Add subject line
   b. Add preview text
   c. Format for email (avoid complex formatting)
   d. Add unsubscribe reminder note
   e. Write to `OUTPUT/publish_[timestamp]/email.md`

   **Facebook:**
   a. Conversational tone
   b. Question or engagement prompt at end
   c. Max 500 chars recommended
   d. Write to `OUTPUT/publish_[timestamp]/facebook.md`

6. **Generate preview document**: Write to `OUTPUT/publish_[timestamp]/preview.md`:
   ```
   # Content Preview - All Platforms

   ## Original Content
   [First 200 chars...]

   ## Platform Adaptations

   ### Twitter
   [Adapted content]
   Character count: [N]

   ### LinkedIn
   [Adapted content]
   Character count: [N]

   ... (for each platform)
   ```

### Phase 3: Validation

7. **Check platform requirements**:
   - Character limits respected
   - No blocked words (check for common spam triggers)
   - Links properly formatted
   - Media references valid

8. **Check content quality**:
   - No spelling errors (basic check)
   - Consistent brand voice
   - Call-to-action present
   - No sensitive information exposed

9. **Create validation report**: Write to `OUTPUT/publish_[timestamp]/validation.md`

10. **Decision point**:
    - IF validation passes: Proceed to publishing
    - IF warnings: Show user and ask for confirmation
    - IF errors: Stop and report issues

### Phase 4: Publishing

11. **Confirm with user** (unless auto-publish enabled):
    - Show summary: "[N] platforms ready, [M] posts total"
    - List any warnings
    - Ask: "Ready to publish? (yes/no/preview first)"

12. **FOR EACH platform** (in order of priority):
    a. Prepare API payload
    b. Make publish request via `http_call`:
       ```
       POST [platform_api_url]
       Headers: Authorization: Bearer [token]
       Body: [adapted_content]
       ```
    c. Record result: success, post ID, URL
    d. IF failure: Log error, continue to next platform
    e. Wait 2 seconds between platforms (rate limiting)

13. **Track published content**: Update `campaign.json`:
    ```json
    {
      "platforms_attempted": 5,
      "platforms_succeeded": 4,
      "platforms_failed": 1,
      "posts": [
        {"platform": "twitter", "status": "success", "url": "...", "id": "..."},
        {"platform": "linkedin", "status": "success", "url": "..."},
        {"platform": "blog", "status": "failed", "error": "..."}
      ]
    }
    ```

### Phase 5: Verification

14. **Verify published content** (for successful posts):
    - Fetch published post via API
    - Confirm content matches
    - Check visibility status

15. **Handle failures**:
    - IF platform failed: Log reason, suggest retry or manual post
    - IF partial success: Report which succeeded/failed
    - Write failure analysis to `OUTPUT/publish_[timestamp]/failures.md`

16. **Retry failed platforms** (if requested):
    - Wait 30 seconds
    - Attempt publish again
    - Update status

### Phase 6: Reporting

17. **Calculate engagement baseline** (if metrics available):
    - Initial view count
    - Current follower counts
    - Note for future comparison

18. **Remember campaign**: Store in memory:
    - "Published campaign [ID] on [DATE] to [N] platforms"
    - "Content topic: [main topic]"
    - "Platform status: [success count]/[total]"

19. **Generate campaign report**: Write to `OUTPUT/publish_[timestamp]/report.md`:
    ```
    # Publishing Campaign Report

    ## Summary
    - Campaign ID: [ID]
    - Published: [DATE TIME]
    - Platforms: [N] attempted, [M] succeeded

    ## Published Posts
    | Platform | Status | URL | Notes |
    |----------|--------|-----|-------|
    | Twitter  | Success | [link] | Thread: 3 posts |
    | LinkedIn | Success | [link] | |
    | Blog     | Failed  | N/A | Error: [reason] |

    ## Content Performance (Initial)
    [To be updated after 24h]

    ## Failed Posts - Action Required
    1. [Platform]: [Error] - [Suggested action]

    ## Files Generated
    - original.md
    - twitter.md
    - linkedin.md
    - ...
    ```

20. **Report to user**:
    - "Campaign [ID] complete: [M]/[N] platforms published successfully"
    - List URLs of successful posts
    - Note any failures with suggested actions
    - Ask: "Want me to retry failed platforms or schedule engagement tracking?"

---

## Error Handling

- IF API rate limited: Wait and retry with exponential backoff
- IF authentication fails: Report which platform, suggest checking credentials
- IF content rejected: Show platform's error message, suggest edits
- IF all platforms fail: Save all adapted content for manual posting
- IF media upload fails: Post without media, note in report

---

## Success Criteria

- [ ] Original content analyzed and saved
- [ ] Platform-specific versions created
- [ ] Validation completed for all platforms
- [ ] Publication attempted on all requested platforms
- [ ] Success/failure recorded for each platform
- [ ] Published URLs collected and reported
- [ ] Memory updated with campaign record
- [ ] User received comprehensive report

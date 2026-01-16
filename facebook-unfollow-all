ROBUST FACEBOOK UNFOLLOW AUTOMATION PROMPT

==================================================
TASK OVERVIEW
==================================================
Systematically unfollow ALL accounts, friends, and pages from a Facebook profile's "Following" tab by visiting each one individually and using the unfollow mechanism, while explicitly ignoring pending friend requests.

==================================================
CORE REQUIREMENTS
==================================================

1. EXCLUSION RULES (CRITICAL)
------------------------------
- NEVER interact with "Cancel request" buttons (these are pending friend requests, not followed accounts)
- Only process entries with:
  * "Friends" button (actual friends being followed)
  * "Following" button (pages/accounts being followed)
  * Page entries without any button status

2. IDENTIFICATION STRATEGY
--------------------------
Target Types to Unfollow:
  - Friends with "Friends" button
  - Pages with "Following" button
  - Public figures with "Following" button
  - Any account/page that shows follow status

Ignore:
  - Any entry with "Cancel request" button

3. SCROLL & LOAD MECHANISM
--------------------------
Progressive scroll strategy:
1. Navigate to /[username]/following tab
2. Scroll to top (cmd+up on Mac or ctrl+up on Windows/Linux)
3. Scroll incrementally with scroll_parameters: {"scroll_amount": "max"}
4. Wait for dynamic content loading (status: "Loading..." element)
5. Use get_page_text to capture full loaded content
6. Repeat until no new content loads (detect by comparing page text)

4. DOM PARSING & VERIFICATION
------------------------------
Step 1: Initial Parse
- Use read_page with filter="interactive" depth=8-10
- Identify ALL interactive buttons: "Friends", "Following", "Cancel request"
- Store list of accounts with actionable buttons (not Cancel request)

Step 2: Smart Filtering
- Extract only entries WITHOUT "Cancel request" in their button text
- Build queue of accounts to process: [{name, url, buttonType}, ...]

Step 3: Verification Before Action
- Click profile/page link
- Wait for page load (2-3 seconds)
- Verify button exists on individual page
- Check if already unfollowed (button shows "Follow" in menu)

5. UNFOLLOW WORKFLOW (PER ACCOUNT)
-----------------------------------
For Friends:
1. Navigate to profile URL
2. Click "Friends" button (coordinates or ref)
3. Wait for menu to appear
4. Look for "Unfollow" option in menu
5. If "Follow" appears → already unfollowed, skip
6. If "Unfollow" appears → click it
7. Click "Update" to confirm (if dialog appears)
8. Verify button state change
9. Navigate back to /following tab

For Pages:
1. Navigate to page URL
2. Click "..." menu OR "Following" button
3. Look for "Following" submenu
4. Click "Unfollow" option
5. Click "Update" to confirm
6. Verify button changes to "Follow"
7. Navigate back to /following tab

6. RETRY MECHANISMS
-------------------
Retry Strategy:
  - Click failure → retry with adjusted coordinates (3 attempts)
  - Menu not appearing → press Escape, retry click (2 attempts)
  - Page load timeout → wait additional 3s, retry navigation (2 attempts)
  - Stale element → re-read page, locate element again (2 attempts)
  - Verification failure → mark as "needs manual review", continue

Recovery Actions:
  - If stuck in menu: Press Escape key
  - If navigation fails: Use navigate tool with direct URL
  - If page crashes: Refresh page, restart from last checkpoint

7. PROGRESS TRACKING
--------------------
Maintain State:
{
  "total_found": 0,
  "successfully_unfollowed": [],
  "already_unfollowed": [],
  "failed": [],
  "pending_requests_ignored": 0,
  "current_index": 0
}

Checkpoint System:
- After every 5 unfollows → log progress
- After every 10 unfollows → verify Following tab state
- Store failed accounts for retry pass

8. ROBUST CHECKS
----------------
Pre-Action Checks:
  - Is "Cancel request" present? → Skip
  - Is button "Friends" or "Following"? → Process
  - Is element visible and clickable? → Proceed
  - Has page finished loading? → Wait if not

Post-Action Checks:
  - Did menu appear after click? → Verify
  - Was "Unfollow" option visible? → Confirm
  - Did button state change after action? → Validate
  - Does profile now show "Follow"? → Success

9. DYNAMIC CONTENT HANDLING
---------------------------
Facebook's Infinite Scroll:
1. Scroll to bottom with scroll_amount="max"
2. Check for status="Loading..." element
3. Wait 1-2 seconds for new content
4. Compare before/after element counts
5. If no new content for 3 consecutive scrolls → reached end
6. Use get_page_text to extract all loaded entries

10. ERROR SCENARIOS & SOLUTIONS
--------------------------------
Error                          | Solution
-------------------------------|------------------------------------------
Menu doesn't open              | Press Escape, wait 1s, retry click
Wrong menu appears             | Press Escape, use find tool to locate correct element
"Unfollow" option missing      | Account already unfollowed, verify and skip
Navigation fails               | Retry with navigate tool, fallback to back button
Element becomes stale          | Re-read page, locate element with fresh reference
Rate limiting detected         | Wait 10-30 seconds, resume
Page layout changes            | Re-scan with read_page, adapt to new structure

11. COMPLETION CRITERIA
-----------------------
Task Complete When:
  - Scrolled entire Following tab (no more content loads)
  - All entries with "Friends"/"Following" buttons processed
  - Only "Cancel request" entries remain
  - Verified 2-3 processed accounts still show "Follow" option
  - get_page_text shows only pending requests in output

12. OPTIMIZATION STRATEGIES
---------------------------
Efficiency Tips:
- Batch read_page calls instead of multiple small queries
- Use find tool only when read_page fails
- Prefer element references (ref) over coordinates
- Cache already-checked accounts to avoid re-processing
- Process accounts in order (top to bottom) to track position

13. EXAMPLE EXECUTION FLOW
--------------------------
1. Navigate to https://www.facebook.com/[USERNAME]/following
2. Press cmd+up (Mac) or ctrl+up (Windows/Linux) to scroll to top
3. Call get_page_text → extract all account names
4. Call read_page filter="interactive" → get all buttons
5. Filter: keep only accounts WITHOUT "Cancel request"
6. FOR EACH filtered account:
   a. Click account name/link
   b. Wait for page load (2s)
   c. Locate Friends/Following button
   d. Click button → open menu
   e. Locate "Unfollow" option
   f. If "Follow" shown → skip (already unfollowed)
   g. If "Unfollow" shown → click it
   h. Click "Update" if dialog appears
   i. Press Escape to close any menus
   j. Navigate back to /following tab
   k. Update progress tracker
7. Scroll down on Following tab
8. Check if new accounts loaded
9. Repeat from step 3 until no more accounts

FINAL: Verify all remaining entries have "Cancel request"

14. SAFETY MEASURES
-------------------
Never:
  - Click "Cancel request" buttons
  - Unfriend people (only unfollow)
  - Modify account settings beyond unfollow
  - Process same account twice
  - Continue if pattern changes unexpectedly

Always:
  - Verify button text before clicking
  - Check for "Cancel request" in element hierarchy
  - Navigate back to Following tab after each action
  - Log all actions taken
  - Pause if errors exceed threshold (>10% failure rate)

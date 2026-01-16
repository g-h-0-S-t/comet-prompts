TAKE CONTROL OF MY BROWSER AND DO THE FOLLOWING - 

═══════════════════════════════════════════════════════════════════════════════
FACEBOOK UNFOLLOW AUTOMATION - SAFEGUARDED EDITION (NEVER UNFRIEND)
═══════════════════════════════════════════════════════════════════════════════

🚨 CRITICAL SAFETY RULES - ABSOLUTE PRIORITY 🚨
-----------------------------------------------
NEVER CLICK ANY BUTTON OR MENU ITEM CONTAINING THESE EXACT STRINGS:
- "Unfriend"
- "Remove friend"
- "Remove from friends"
- "Delete friend"
- "Unfriend [name]"

IF ANY OF THESE STRINGS ARE DETECTED IN A MENU:
1. Immediately press Escape to close the menu
2. Mark account as "FAILED" with reason "UNFRIEND_DETECTED"
3. Do NOT click anything
4. Return to following list
5. Skip to next account

ONLY VALID CLICK TARGETS:
- "Unfollow" (exact match or contains "Unfollow" but NOT "Unfriend")
- "Following" (submenu to access Unfollow)
- "Friends" button (to open dropdown, NOT to unfriend)

BEFORE EVERY CLICK IN A MENU:
1. Read the exact text of the element you're about to click
2. Verify it matches ONLY "Unfollow" or "Following"
3. If text contains "Unfriend" → ABORT and close menu

═══════════════════════════════════════════════════════════════════════════════
UNDERSTANDING THE DIFFERENCE
═══════════════════════════════════════════════════════════════════════════════

UNFOLLOW (SAFE - what we want):
- Stops posts from appearing in your feed
- KEEPS the friendship intact
- Person remains in your friends list
- They don't get notified
- Button changes from "Friends" → still "Friends" but with a different follow state

UNFRIEND (DANGEROUS - what we must avoid):
- REMOVES the person from your friends list completely
- BREAKS the friendship connection
- Requires sending a new friend request to reconnect
- This is PERMANENT and cannot be undone

═══════════════════════════════════════════════════════════════════════════════
ROLE & CONSTRAINTS
═══════════════════════════════════════════════════════════════════════════════

You are a strict browser-automation agent controlling ONE active tab.
Target: UNFOLLOW people (stop seeing their posts) WITHOUT unfriending them.

Core behavior:
- Use only: navigate, scroll, key_press, read_page, get_page_text, click
- NEVER execute JavaScript directly
- Process ONE account at a time sequentially
- ALWAYS return to following list between accounts
- NEVER open multiple tabs

Target URL: https://www.facebook.com/[USERNAME]/following

═══════════════════════════════════════════════════════════════════════════════
PHASE 1: LOAD COMPLETE FOLLOWING LIST
═══════════════════════════════════════════════════════════════════════════════

1. Confirm URL contains "/following", otherwise navigate to:
   https://www.facebook.com/[USERNAME]/following
   Wait 3 seconds.

2. Scroll to absolute top:
   - Mac: cmd+up
   - Windows/Linux: ctrl+up

3. Infinite scroll to load ALL entries:

   previous_content = ""
   no_change_count = 0

   WHILE no_change_count < 3:
     - scroll down with amount="max"
     - wait 2-3 seconds
     - current_content = get_page_text()
     - IF current_content == previous_content:
         no_change_count += 1
       ELSE:
         no_change_count = 0
     - previous_content = current_content
   END WHILE

═══════════════════════════════════════════════════════════════════════════════
PHASE 2: DISCOVER ACCOUNTS & BUILD QUEUE
═══════════════════════════════════════════════════════════════════════════════

Execute: read_page(filter="interactive", depth=12)

Extract for each entry:
- account_name
- account_url
- button_text (e.g., "Friends", "Following", "Cancel request")
- element_ref
- position_index

BUILD QUEUE:

processing_queue = []
skipped_pending = 0

FOR each entry:
  IF button_text contains "Cancel request":
    skipped_pending += 1
    CONTINUE  # Skip pending friend requests
  
  ELSE IF button_text == "Friends" OR button_text == "Following":
    ADD to processing_queue:
      {
        name: account_name,
        url: account_url,
        button_type: button_text,
        ref: element_ref,
        status: "pending",
        attempts: 0,
        index: position_index
      }

═══════════════════════════════════════════════════════════════════════════════
PHASE 3: MAIN PROCESSING LOOP
═══════════════════════════════════════════════════════════════════════════════

FOR each account in processing_queue:
  IF account.status == "unfollowed" OR "already_unfollowed":
    SKIP
  
  result = process_single_account(account)
  
  UPDATE account.status based on result:
    - "SUCCESS" → account.status = "unfollowed"
    - "ALREADY_UNFOLLOWED" → account.status = "already_unfollowed"
    - "FAILED" → account.attempts += 1
    - "UNFRIEND_DETECTED" → account.status = "blocked_for_safety"
  
  Navigate back to: https://www.facebook.com/[USERNAME]/following
  Wait 3 seconds

═══════════════════════════════════════════════════════════════════════════════
FUNCTION: process_single_account(account)
═══════════════════════════════════════════════════════════════════════════════

OUTPUT: "SUCCESS" | "ALREADY_UNFOLLOWED" | "FAILED" | "UNFRIEND_DETECTED"

─────────────────────────────────────────────────────────────────────────────
A. NAVIGATE TO ACCOUNT PAGE
─────────────────────────────────────────────────────────────────────────────
1. navigate(account.url)
2. wait 3 seconds

Pre-flight checks:
- Verify page loaded (not "Page not available")
- Retry up to 2 times if failed
- If still fails → RETURN "FAILED"

─────────────────────────────────────────────────────────────────────────────
B. DETECT CURRENT FOLLOW STATUS
─────────────────────────────────────────────────────────────────────────────
Execute: read_page(filter="interactive", depth=10)

Find buttons:
- primary_button:
    IF account.button_type == "Friends" → button "Friends"
    IF account.button_type == "Following" → button "Following"
- fallback_follow_button: button "Follow"

IF fallback_follow_button exists AND primary_button does NOT:
  → RETURN "ALREADY_UNFOLLOWED"

─────────────────────────────────────────────────────────────────────────────
C. OPEN ACTION MENU (WITH SAFETY CHECK)
─────────────────────────────────────────────────────────────────────────────
IF primary_button NOT found:
  Try three-dot menu "..."
  IF found:
    click it, wait 1.5 seconds, read_page
  ELSE:
    RETURN "FAILED"

ELSE:
  click(primary_button)
  wait 1 second

Wait 1.5 seconds
Execute: read_page(filter="interactive", depth=8)

─────────────────────────────────────────────────────────────────────────────
D. SAFETY CHECK - SCAN MENU FOR UNFRIEND
─────────────────────────────────────────────────────────────────────────────
🚨 CRITICAL SAFETY STEP 🚨

Get all menu item texts from the page read.

FOR each menu_item in visible_menu:
  item_text_lower = menu_item.text.lower()
  
  IF any of these in item_text_lower:
    - "unfriend"
    - "remove friend"
    - "delete friend"
    - "remove from friend"
  
  THEN:
    Press Escape immediately
    RETURN "UNFRIEND_DETECTED"

If this check passes, proceed to E.

─────────────────────────────────────────────────────────────────────────────
E. LOCATE UNFOLLOW OPTION (WITH PRE-CLICK VERIFICATION)
─────────────────────────────────────────────────────────────────────────────
Search menu for:
- unfollow_option = item containing "Unfollow" (case-insensitive)
- follow_option = item containing "Follow" but NOT "Unfollow"
- following_submenu = item "Following"

VERIFICATION BEFORE PROCEEDING:
  IF unfollow_option found:
    option_text = unfollow_option.text.lower()
    
    🚨 DOUBLE-CHECK: Does option_text contain "unfriend"?
    IF YES:
      Press Escape
      RETURN "UNFRIEND_DETECTED"
    
    IF NO:
      # Safe to proceed

IF follow_option exists AND unfollow_option does NOT:
  Press Escape
  RETURN "ALREADY_UNFOLLOWED"

IF unfollow_option exists directly:
  PROCEED to F

ELSE IF following_submenu exists:
  click(following_submenu)
  wait 1 second
  read_page(depth=8)
  
  # Run safety check again on submenu
  FOR each submenu_item:
    IF "unfriend" in submenu_item.text.lower():
      Press Escape
      RETURN "UNFRIEND_DETECTED"
  
  unfollow_option = find("Unfollow")
  IF NOT found:
    Press Escape
    RETURN "FAILED"

ELSE:
  Press Escape
  RETURN "FAILED"

─────────────────────────────────────────────────────────────────────────────
F. EXECUTE UNFOLLOW (WITH FINAL SAFETY CHECK)
─────────────────────────────────────────────────────────────────────────────
🚨 FINAL VERIFICATION BEFORE CLICK 🚨

option_text = unfollow_option.text.lower()

IF "unfriend" in option_text OR "remove friend" in option_text:
  Press Escape
  RETURN "UNFRIEND_DETECTED"

IF "unfollow" NOT in option_text:
  Press Escape
  RETURN "FAILED"

# Safe to click
click(unfollow_option)
wait 1 second

Check for confirmation dialog:
  read_page(filter="interactive", depth=6)
  confirmation_button = find(["Update", "Confirm", "Unfollow"])
  
  IF found:
    🚨 VERIFY confirmation button text does NOT contain "Unfriend"
    IF "unfriend" in confirmation_button.text.lower():
      Press Escape
      RETURN "UNFRIEND_DETECTED"
    
    click(confirmation_button)
    wait 1 second

Press Escape
wait 1 second

─────────────────────────────────────────────────────────────────────────────
G. VERIFY UNFOLLOW SUCCESS
─────────────────────────────────────────────────────────────────────────────
read_page(filter="interactive", depth=10)

Find:
- follow_button = button "Follow"
- friends_button = button "Friends"
- following_button = button "Following"

🚨 CRITICAL: Check that Friends button still exists
IF friends_button does NOT exist AND account.button_type was "Friends":
  # This might indicate unfriending happened
  RETURN "FAILED"  # Mark for manual review

IF follow_button exists:
  RETURN "SUCCESS"
ELSE IF friends_button OR following_button visible:
  RETURN "FAILED"
ELSE:
  RETURN "FAILED"

═══════════════════════════════════════════════════════════════════════════════
PHASE 4: RETRY & RECOVERY
═══════════════════════════════════════════════════════════════════════════════

Retry only accounts with status "pending" AND attempts < 3.

RETRY PASS 1:
  FOR each account in retry_queue:
    wait 3 seconds
    result = process_single_account(account)
    update status and attempts
    navigate back to following list

RETRY PASS 2:
  FOR accounts still pending with attempts < 3:
    wait 5 seconds
    result = process_single_account(account)
    update status
    navigate back

Accounts with "UNFRIEND_DETECTED" or attempts >= 3:
  Mark as "manual_review_needed"

═══════════════════════════════════════════════════════════════════════════════
PHASE 5: VERIFICATION SWEEP
═══════════════════════════════════════════════════════════════════════════════

1. Navigate to: https://www.facebook.com/[USERNAME]/following
2. Scroll to top
3. Re-run infinite scroll loader
4. read_page(filter="interactive", depth=12)

5. Count:
   - remaining_friends_buttons = count("Friends")
   - remaining_following_buttons = count("Following")
   - total_remaining = remaining_friends_buttons + remaining_following_buttons

6. IF total_remaining == 0:
     STOP - All done

7. IF total_remaining > 0:
     Extract new accounts
     Filter out already processed
     Re-process with same safety checks
     Run verification sweep again (recursive)

═══════════════════════════════════════════════════════════════════════════════
FINAL SAFETY SUMMARY
═══════════════════════════════════════════════════════════════════════════════

Every click goes through:
1. Text extraction
2. "unfriend" keyword scan (case-insensitive)
3. Abort if detected
4. Only proceed if text contains "unfollow" or "following"

Any account triggering "UNFRIEND_DETECTED":
- Immediately abort
- Close menu
- Mark for manual review
- Never click

This ensures friendships remain 100% intact while only removing follows.

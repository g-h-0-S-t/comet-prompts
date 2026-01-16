TAKE CONTROL OF MY BROWSER AND DO THE FOLLOWING - 

═══════════════════════════════════════════════════════════════════════════════
FACEBOOK UNFOLLOW-ONLY AUTOMATION (ABSOLUTE UNFRIEND PROTECTION)
═══════════════════════════════════════════════════════════════════════════════

🔴 CRITICAL RULE #1: NEVER REMOVE FRIENDSHIPS 🔴
-------------------------------------------------
THIS AUTOMATION MUST ONLY CLICK "UNFOLLOW" - NOTHING ELSE

UNFOLLOW = Remove from news feed (SAFE - keeps friendship)
UNFRIEND = Remove from friends list (DANGEROUS - breaks connection)

🔴 ABSOLUTE PROHIBITION 🔴
IF YOU SEE ANY OF THESE WORDS IN A MENU, DO NOT CLICK ANYTHING:
- "Unfriend"
- "Remove friend" 
- "Delete friend"
- "Block"
- "Restrict"

ONLY CLICK IF THE TEXT SAYS EXACTLY:
- "Unfollow" (and nothing else)
- "Unfollow [Name]" (and nothing else)

IF THERE IS ANY DOUBT WHATSOEVER:
1. Press Escape
2. Skip this account
3. Move to next account
4. NEVER CLICK WHEN UNSURE

═══════════════════════════════════════════════════════════════════════════════
HOW TO IDENTIFY THE CORRECT OPTION
═══════════════════════════════════════════════════════════════════════════════

WHEN YOU CLICK THE "FRIENDS" BUTTON, A MENU APPEARS WITH OPTIONS:

✅ CORRECT OPTION (SAFE TO CLICK):
   Text: "Unfollow [Name]"
   Icon: Usually has a bell or news feed icon
   Effect: Stops their posts from appearing in your feed
   Result: They remain in your friends list

❌ WRONG OPTION (NEVER CLICK):
   Text: "Unfriend [Name]" 
   Icon: Usually has an "X" or person icon
   Effect: REMOVES them from your friends list completely
   Result: Friendship is destroyed

🔴 THESE ARE DIFFERENT OPTIONS IN THE SAME MENU 🔴
🔴 YOU MUST CLICK ONLY "UNFOLLOW" - NEVER "UNFRIEND" 🔴

═══════════════════════════════════════════════════════════════════════════════
SAFE CLICKING PROTOCOL
═══════════════════════════════════════════════════════════════════════════════

STEP 1: Open the menu
- Click "Friends" button on profile
- Wait 1.5 seconds

STEP 2: Read ALL menu items
- Execute: read_page(filter="interactive", depth=8)
- Get the EXACT text of EVERY menu item

STEP 3: Identify safe target
- Look for menu item with EXACT text: "Unfollow" or "Unfollow [Name]"
- Store its reference

STEP 4: Verify safe target (CRITICAL)
- Take the text of the menu item you're about to click
- Convert to lowercase
- Check if it contains "unfriend" → IF YES: Press Escape, RETURN "BLOCKED"
- Check if it contains "remove friend" → IF YES: Press Escape, RETURN "BLOCKED"  
- Check if it contains "block" → IF YES: Press Escape, RETURN "BLOCKED"
- Check if it contains "unfollow" → IF YES: Safe to proceed
- IF NO "unfollow" found → Press Escape, RETURN "FAILED"

STEP 5: Click only if verified safe
- Click the verified "Unfollow" option
- Wait 1 second
- Handle any confirmation dialog (verify it also says "Unfollow")

STEP 6: Close menu and verify
- Press Escape
- Verify the "Friends" button still exists (means friendship intact)

═══════════════════════════════════════════════════════════════════════════════
ROLE & CONSTRAINTS  
═══════════════════════════════════════════════════════════════════════════════

You are a browser automation agent.
Target: UNFOLLOW people WITHOUT unfriending them.

Constraints:
- Process ONE account at a time
- ALWAYS return to following list between accounts  
- Use only: navigate, scroll, read_page, get_page_text, click
- NEVER click anything containing "unfriend"

Target URL: https://www.facebook.com/[USERNAME]/following

═══════════════════════════════════════════════════════════════════════════════
PHASE 1: LOAD FOLLOWING LIST
═══════════════════════════════════════════════════════════════════════════════

1. Navigate to: https://www.facebook.com/[USERNAME]/following
   Wait 3 seconds

2. Scroll to top:
   - Mac: cmd+up
   - Windows/Linux: ctrl+up

3. Load all entries with infinite scroll:

   previous_content = ""
   no_change_count = 0

   WHILE no_change_count < 3:
     scroll down (amount="max")
     wait 3 seconds
     current_content = get_page_text()
     IF current_content == previous_content:
       no_change_count += 1
     ELSE:
       no_change_count = 0
     previous_content = current_content
   END WHILE

═══════════════════════════════════════════════════════════════════════════════
PHASE 2: BUILD QUEUE
═══════════════════════════════════════════════════════════════════════════════

Execute: read_page(filter="interactive", depth=12)

FOR each entry with button "Friends" OR "Following":
  ADD to queue:
    {
      name: account_name,
      url: account_url,
      button_type: button_text,
      status: "pending",
      attempts: 0
    }

SKIP entries with "Cancel request" button

═══════════════════════════════════════════════════════════════════════════════
PHASE 3: PROCESS EACH ACCOUNT
═══════════════════════════════════════════════════════════════════════════════

FOR each account in queue:
  result = process_account(account)
  
  Navigate back to: https://www.facebook.com/[USERNAME]/following
  Wait 3 seconds
  
  Continue to next account

═══════════════════════════════════════════════════════════════════════════════
FUNCTION: process_account(account)
═══════════════════════════════════════════════════════════════════════════════

STEP 1: Navigate to profile
----------------------------
navigate(account.url)
wait 3 seconds

IF page failed to load:
  RETURN "FAILED"

STEP 2: Find follow status
---------------------------
read_page(filter="interactive", depth=10)

Find:
- friends_button = button labeled "Friends"
- following_button = button labeled "Following"  
- follow_button = button labeled "Follow"

IF follow_button exists AND (friends_button OR following_button) does NOT exist:
  # Already unfollowed
  RETURN "ALREADY_UNFOLLOWED"

STEP 3: Open menu
-----------------
IF account.button_type == "Friends":
  target_button = friends_button
ELSE IF account.button_type == "Following":
  target_button = following_button
ELSE:
  RETURN "FAILED"

IF target_button NOT found:
  RETURN "FAILED"

click(target_button)
wait 1.5 seconds

STEP 4: Read menu and find safe option
---------------------------------------
read_page(filter="interactive", depth=8)

safe_unfollow_option = null

FOR each menu_item in visible menu items:
  item_text = menu_item.text
  item_text_lower = item_text.lower()
  
  🔴 SAFETY CHECK: Skip dangerous options
  IF "unfriend" in item_text_lower:
    CONTINUE to next item (DO NOT store this)
  IF "remove friend" in item_text_lower:
    CONTINUE to next item
  IF "block" in item_text_lower:
    CONTINUE to next item
  
  ✅ SAFE CHECK: Identify unfollow option
  IF "unfollow" in item_text_lower:
    safe_unfollow_option = menu_item
    BREAK  # Found it

IF safe_unfollow_option is null:
  # No safe unfollow option found
  Press Escape
  RETURN "FAILED"

STEP 5: Verify before clicking
-------------------------------
🔴 FINAL VERIFICATION 🔴

option_text = safe_unfollow_option.text.lower()

# Triple-check it's safe
IF "unfriend" in option_text:
  Press Escape
  RETURN "BLOCKED_DANGEROUS"

IF "unfollow" NOT in option_text:
  Press Escape  
  RETURN "FAILED"

# Confirmed safe - proceed
click(safe_unfollow_option)
wait 1 second

STEP 6: Handle confirmation (if any)
-------------------------------------
read_page(filter="interactive", depth=6)

confirmation = find_button(["Unfollow", "Confirm", "Update"])

IF confirmation found:
  conf_text = confirmation.text.lower()
  
  # Verify confirmation is safe
  IF "unfriend" in conf_text:
    Press Escape
    RETURN "BLOCKED_DANGEROUS"
  
  IF "unfollow" in conf_text OR "confirm" in conf_text OR "update" in conf_text:
    click(confirmation)
    wait 1 second

Press Escape
wait 1 second

STEP 7: Verify success
-----------------------
read_page(filter="interactive", depth=10)

Find:
- follow_button = button "Follow"
- friends_button = button "Friends"

🔴 CRITICAL: Verify friendship still intact
IF account.button_type was "Friends":
  IF friends_button does NOT exist:
    # Friends button disappeared - might have unfriended by mistake
    RETURN "ERROR_FRIENDSHIP_LOST"

IF follow_button exists:
  RETURN "SUCCESS"
ELSE:
  RETURN "FAILED"

═══════════════════════════════════════════════════════════════════════════════
PHASE 4: RETRY FAILED ACCOUNTS
═══════════════════════════════════════════════════════════════════════════════

Retry accounts with "FAILED" status (attempts < 3)

FOR each failed account:
  wait 3 seconds
  result = process_account(account)
  update status
  navigate back to following list

Accounts with "BLOCKED_DANGEROUS" or "ERROR_FRIENDSHIP_LOST":
  DO NOT RETRY - mark for manual review

═══════════════════════════════════════════════════════════════════════════════
PHASE 5: VERIFICATION
═══════════════════════════════════════════════════════════════════════════════

1. Navigate to: https://www.facebook.com/[USERNAME]/following
2. Scroll to top
3. Re-run infinite scroll loader  
4. read_page(filter="interactive", depth=12)

Count remaining:
- friends_buttons = count("Friends")
- following_buttons = count("Following")
- total = friends_buttons + following_buttons

IF total == 0:
  STOP - Complete

IF total > 0:
  Extract new accounts
  Filter already processed
  Process again with same safety protocol

═══════════════════════════════════════════════════════════════════════════════
EMERGENCY STOP CONDITIONS
═══════════════════════════════════════════════════════════════════════════════

IMMEDIATELY STOP ALL AUTOMATION IF:
1. Any account returns "ERROR_FRIENDSHIP_LOST"
2. More than 5 accounts return "BLOCKED_DANGEROUS"  
3. Any menu contains "Unfriend" as the only option

These indicate the automation might be clicking wrong options.

═══════════════════════════════════════════════════════════════════════════════
SUMMARY OF SAFETY PROTOCOL
═══════════════════════════════════════════════════════════════════════════════

✅ Only click menu items containing "unfollow"
❌ Never click menu items containing "unfriend"
🔴 When in doubt, skip the account
🔴 Verify "Friends" button still exists after action
🔴 Stop immediately if friendships are being lost

This ensures ZERO chance of unfriending anyone.

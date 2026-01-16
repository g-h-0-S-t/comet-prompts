TAKE CONTROL OF MY BROWSER AND DO THE FOLLOWING - 

═══════════════════════════════════════════════════════════════════════════════
ENHANCED FACEBOOK UNFOLLOW AUTOMATION PROMPT - RECURSIVE EDITION
═══════════════════════════════════════════════════════════════════════════════

ROLE & CONSTRAINTS
------------------
You are a strict browser-automation agent controlling ONE active tab.
You must:
- Use only these actions: navigate, scroll, key_press, read_page, get_page_text, click.
- NEVER execute JavaScript directly in the console.
- ALWAYS operate sequentially: handle ONE account fully (open → unfollow → verify → back to list), then move on to the next.
- NEVER open additional tabs or windows.
- ALWAYS return to the main following list URL after processing each account.

Target URL pattern for the following list:
- https://www.facebook.com/[USERNAME]/following

Replace [USERNAME] with the actual username that is already open in the active tab.

Your CORE LOOP is:
1) Ensure the full following list is loaded.
2) Build a queue of accounts to process.
3) For each account in the queue:
   - Open its profile/page.
   - Unfollow with verification + retries.
   - Navigate back to the following list.
   - Continue from the next account.
4) When no actionable “Friends” / “Following” remain, stop.

═══════════════════════════════════════════════════════════════════════════════
PHASE 1: LOAD COMPLETE FOLLOWING LIST
═══════════════════════════════════════════════════════════════════════════════

1. Confirm you are on the following page:
   - If URL does NOT contain "/following", navigate to:
     - https://www.facebook.com/[USERNAME]/following
   - Wait ~3 seconds for full load.

2. Scroll to absolute top:
   - key_press: "cmd+up" (Mac) OR "ctrl+up" (Windows/Linux).

3. Infinite scroll loader to load ALL entries:

   - previous_content = ""
   - no_change_count = 0

   WHILE no_change_count < 3:
     - scroll(type="page_down", amount="max")
     - wait 2–3 seconds
     - current_content = get_page_text()
     - IF current_content == previous_content:
         no_change_count += 1
       ELSE:
         no_change_count = 0
     - previous_content = current_content
   END WHILE

   After this loop, assume the full following list is visible in the DOM.

═══════════════════════════════════════════════════════════════════════════════
PHASE 2: DISCOVER ACCOUNTS & BUILD QUEUE
═══════════════════════════════════════════════════════════════════════════════

Use read_page(filter="interactive", depth=12).

For each visible entry in the following list, extract:
- account_name (text near the button, usually a clickable name)
- account_url (href of the clickable name)
- button_text (e.g., "Friends", "Following", "Cancel request")
- element_ref (reference to the button element)
- position_index (0-based index in the current list order)

PSEUDOCODE:

processing_queue = []
pending_requests_count = 0

FOR each discovered entry:
  IF button_text contains "Cancel request":
    pending_requests_count += 1
    CONTINUE  # skip pending requests completely
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
  ELSE:
    CONTINUE  # ignore anything else

Result: processing_queue contains ONLY accounts that should be unfollowed.

═══════════════════════════════════════════════════════════════════════════════
PHASE 3: MAIN RECURSIVE PROCESSING LOOP
═══════════════════════════════════════════════════════════════════════════════

You MUST process accounts ONE BY ONE in this order:
- For i from 0 to processing_queue.length - 1:
  - account = processing_queue[i]
  - If account.status is already "unfollowed" or "already_unfollowed", skip.
  - result = process_single_account(account)
  - UPDATE account.status based on result
  - After finishing, ALWAYS navigate back to the following list:
      navigate("https://www.facebook.com/[USERNAME]/following")
      wait ~3 seconds
      (Do NOT rebuild the queue unless in Phase 4 verification.)

If process_single_account returns:
- "SUCCESS"          → account.status = "unfollowed"
- "ALREADY_UNFOLLOWED" → account.status = "already_unfollowed"
- "FAILED"           → account.attempts += 1
                         if attempts < 3, keep as "pending" for later retry
                         else account.status = "failed_max_retries"

Track:
- successfully_unfollowed = []
- already_unfollowed = []
- failed_accounts = []
- retry_round = 0

After first FULL pass through the queue, run RETRY passes only on accounts with status still "pending" AND attempts < 3.

═══════════════════════════════════════════════════════════════════════════════
FUNCTION: process_single_account(account)
═══════════════════════════════════════════════════════════════════════════════

INPUT: {name, url, button_type, ref, status, attempts}
OUTPUT: "SUCCESS" | "ALREADY_UNFOLLOWED" | "FAILED"

A. NAVIGATE TO ACCOUNT PAGE
---------------------------
1. navigate(account.url)
2. wait ~3 seconds

Pre-flight checks using read_page(filter="all", depth=6):
- If address bar URL clearly does NOT match account.url domain/profile → one retry.
- If page shows "Page not available" / "Content not found" / empty main content:
  - Retry navigate up to 2 times total.
  - If still failing → RETURN "FAILED".

B. DETECT CURRENT FOLLOW STATUS
-------------------------------
1. read_page(filter="interactive", depth=10)

2. Define:
   - primary_button:
       if account.button_type == "Friends"   → button labeled "Friends"
       if account.button_type == "Following" → button labeled "Following"
   - fallback_follow_button: button labeled "Follow"

3. Logic:
   - If fallback_follow_button EXISTS and primary_button does NOT:
       → account is already unfollowed
       → RETURN "ALREADY_UNFOLLOWED"

C. OPEN ACTION MENU
-------------------
1. If primary_button is NOT found:
   - Try locate a three-dot menu "..." near the profile header.
   - If found, click it once, wait ~1.5 seconds, then read_page again.
   - If not found → RETURN "FAILED".

2. If primary_button IS found:
   - Attempt up to 3 clicks:
       - click(primary_button)
       - wait ~1 second
       - If no menu appears after 3 attempts → RETURN "FAILED".

3. After click, wait ~1.5 seconds and read_page(filter="interactive", depth=8) to capture menu options.

D. LOCATE UNFOLLOW OPTION
-------------------------
Search the newly appeared menu content for:
- "Unfollow"
- "Unfollow [account.name]" (case-insensitive)
- "Following" sub-menu that leads to an Unfollow entry

Variables:
- unfollow_option = first menu item with label containing "Unfollow"
- follow_option   = menu item with label containing "Follow"
- following_submenu = menu item exactly or starting with "Following"

Logic:
- If follow_option exists AND unfollow_option does NOT:
    # Menu is offering to Follow → already unfollowed
    - Close menu (Esc key) and RETURN "ALREADY_UNFOLLOWED"

- If unfollow_option exists directly:
    - Proceed to E.

- Else if following_submenu exists:
    - click(following_submenu)
    - wait 1 second
    - read_page(depth=8) again
    - search again for unfollow_option
    - If still not found:
        - close menu (Esc key) and RETURN "FAILED"

- If no relevant option:
    - close menu (Esc key) and RETURN "FAILED"

E. EXECUTE UNFOLLOW
-------------------
1. click(unfollow_option)
2. wait ~1 second

3. Check for confirmation dialog:
   - read_page(filter="interactive", depth=6)
   - Look for confirmation buttons labeled:
       - "Update"
       - "Confirm"
       - "Unfollow"
   - If found, click the confirmation button and wait ~1 second.

4. Press Escape once to close any remaining overlays/menus.
5. Wait ~1 second.

F. VERIFY UNFOLLOW
------------------
1. read_page(filter="interactive", depth=10)

2. Find:
   - follow_button    = button labeled "Follow" or "Follow [account.name]"
   - friends_button   = button labeled "Friends"
   - following_button = button labeled "Following"

3. Verification:
   - If follow_button exists:
       → RETURN "SUCCESS"
   - Else if friends_button OR following_button still visible:
       → RETURN "FAILED"
   - Else:
       → ambiguous → RETURN "FAILED"

G. RETURN STATUS
----------------
Return the status string from step F.

═══════════════════════════════════════════════════════════════════════════════
PHASE 4: RETRY & RECOVERY (RECURSIVE)
═══════════════════════════════════════════════════════════════════════════════

After first pass:
- Collect accounts with status "pending" AND attempts < 3 into retry_queue.

RETRY PASS 1:
- If retry_queue NOT empty:
  - For each account in retry_queue:
    - wait ~3 seconds
    - result = process_single_account(account)
    - update status and attempts
    - ALWAYS navigate back to:
        https://www.facebook.com/[USERNAME]/following
      and wait ~3 seconds before next account.

RETRY PASS 2:
- Build a new retry_queue again with those still "pending" and attempts < 3.
- If still not empty:
  - For each account:
    - wait ~5 seconds
    - result = process_single_account(account)
    - update status and attempts
    - ALWAYS navigate back to the following list after each account.

Any accounts that continue to fail after 2 retry rounds and attempts >= 3:
- Mark status = "failed_max_retries".
- Add to manual_review_list.

═══════════════════════════════════════════════════════════════════════════════
PHASE 5: VERIFICATION SWEEP & RECURSIVE RE-RUN
═══════════════════════════════════════════════════════════════════════════════

1. Navigate to:
   - https://www.facebook.com/[USERNAME]/following
   - wait ~3 seconds
   - scroll to top (cmd+up / ctrl+up)

2. Re-run the infinite scroll loader from PHASE 1.

3. read_page(filter="interactive", depth=12).

4. Count remaining actionable accounts:
   - remaining_friends_buttons   = count of buttons labeled "Friends"
   - remaining_following_buttons = count of buttons labeled "Following"
   - remaining_actionable = remaining_friends_buttons + remaining_following_buttons

5. If remaining_actionable == 0:
   - Stop automation. Only "Cancel request" items should remain.

6. If remaining_actionable > 0:
   - Extract new actionable accounts into new_queue (same rules as PHASE 2).
   - Filter out accounts already in successfully_unfollowed (by URL or name).
   - If new_queue NOT empty:
       - Process each account with process_single_account.
       - After finishing the batch, run this verification sweep (PHASE 5) again.
       - This defines the RECURSIVE behavior: verify → process remaining → verify again.

═══════════════════════════════════════════════════════════════════════════════
PHASE 6: OPTIONAL SPOT CHECK
═══════════════════════════════════════════════════════════════════════════════

Optionally, select a small random subset (e.g., up to 5) from successfully_unfollowed and:

FOR EACH sample_account:
  - navigate(sample_account.url)
  - wait ~2 seconds
  - read_page(filter="interactive", depth=10)
  - Confirm the presence of a "Follow" button and absence of "Friends"/"Following".

If any anomalies appear, add them to anomaly_list for manual review.

═══════════════════════════════════════════════════════════════════════════════

MANDATORY BEHAVIOR SUMMARY (TO ENFORCE CORRECT EXECUTION)
---------------------------------------------------------
- Process accounts strictly SEQUENTIALLY.
- For each account:
  1) From following list → open profile
  2) Unfollow with verification and retries
  3) Navigate BACK to the following list URL
  4) Move to the NEXT account
- Do NOT stay on the profile and search for other accounts.
- Do NOT open multiple tabs.
- Do NOT skip navigation back to the following list between accounts.
- REPEAT this cycle until no “Friends” or “Following” buttons remain on the following page.

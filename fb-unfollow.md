═══════════════════════════════════════════════════════════════════════════════
ENHANCED FACEBOOK UNFOLLOW AUTOMATION PROMPT - RECURSIVE EDITION
═══════════════════════════════════════════════════════════════════════════════

MISSION: Systematically unfollow ALL accounts, friends, and pages from a Facebook 
profile by opening each account individually, verifying follow status, and 
executing unfollow actions with full verification and recursive retry logic.

═══════════════════════════════════════════════════════════════════════════════
PHASE 1: COMPLETE DISCOVERY & LIST BUILDING
═══════════════════════════════════════════════════════════════════════════════

STEP 1.1: NAVIGATE AND LOAD ALL CONTENT
----------------------------------------
1. Navigate to: https://www.facebook.com/[USERNAME]/following
2. Execute: cmd+up (Mac) or ctrl+up (Windows/Linux) to scroll to absolute top
3. Implement infinite scroll loader:
   
   SET previous_content = ""
   SET no_change_count = 0
   
   WHILE no_change_count < 3:
     - Scroll with scroll_amount="max" down
     - Wait 2-3 seconds for dynamic loading
     - Call get_page_text → save as current_content
     - Look for "Loading..." status indicator
     - IF current_content == previous_content:
         no_change_count += 1
       ELSE:
         no_change_count = 0
     - previous_content = current_content
   
   END WHILE
   
   Result: Entire following list is now loaded

STEP 1.2: EXTRACT ALL ACCOUNTS WITH METADATA
---------------------------------------------
Execute: read_page(filter="interactive", depth=12)

For EACH discovered element, extract:
  - Account name (text content)
  - Profile/Page URL (href attribute)
  - Button type ("Friends" / "Following" / "Cancel request")
  - Element reference (ref_id)
  - Position index (for tracking)

STEP 1.3: BUILD PROCESSING QUEUE (CRITICAL FILTERING)
------------------------------------------------------
Initialize: processing_queue = []

FOR EACH account IN discovered_accounts:
  
  # CRITICAL: Identify and skip pending friend requests
  IF button_text CONTAINS "Cancel request":
    INCREMENT pending_requests_count
    SKIP this account (DO NOT ADD TO QUEUE)
    CONTINUE to next account
  
  # Process only accounts with follow relationship
  ELSE IF button_text IN ["Friends", "Following"]:
    ADD TO processing_queue: {
      "name": account_name,
      "url": account_url,
      "button_type": button_text,
      "ref": element_ref,
      "status": "pending",
      "attempts": 0,
      "index": position_number
    }
  
END FOR

OUTPUT: processing_queue contains ONLY accounts to unfollow

═══════════════════════════════════════════════════════════════════════════════
PHASE 2: RECURSIVE ACCOUNT PROCESSING ENGINE
═══════════════════════════════════════════════════════════════════════════════

MAIN PROCESSING LOOP
--------------------
FOR EACH account IN processing_queue:
  
  result = process_single_account(account)
  
  IF result == "SUCCESS":
    account.status = "unfollowed"
    successfully_unfollowed.append(account)
  
  ELSE IF result == "ALREADY_UNFOLLOWED":
    account.status = "already_unfollowed"
    already_unfollowed.append(account)
  
  ELSE IF result == "FAILED":
    IF account.attempts < 3:
      account.attempts += 1
      retry_queue.append(account)
    ELSE:
      account.status = "failed_max_retries"
      failed_accounts.append(account)
  
  # Progress logging
  IF (processed_count % 5) == 0:
    PRINT progress_report()
  
  # Checkpoint verification
  IF (processed_count % 10) == 0:
    verify_following_tab_state()

END FOR

═══════════════════════════════════════════════════════════════════════════════
FUNCTION: process_single_account(account)
═══════════════════════════════════════════════════════════════════════════════

This function processes ONE account completely and returns status.

INPUT: account object with {name, url, button_type, ref, status, attempts}
OUTPUT: "SUCCESS" | "ALREADY_UNFOLLOWED" | "FAILED"

─────────────────────────────────────────────────────────────────────────────
STEP A: NAVIGATE TO ACCOUNT PAGE
─────────────────────────────────────────────────────────────────────────────
1. Execute: navigate(url=account.url, tab_id=current_tab)
2. Wait 3 seconds for complete page load
3. Take screenshot for visual verification (optional debugging)

Pre-flight checks:
  - Verify URL loaded correctly (check browser address bar)
  - Check for error messages ("Page not available", "Content not found")
  - Confirm page content loaded (not blank/loading spinner)

IF page_load_failed:
  RETRY navigation up to 2 more times
  IF still fails:
    RETURN "FAILED"

─────────────────────────────────────────────────────────────────────────────
STEP B: VERIFY ACCOUNT LOADED & IDENTIFY CURRENT STATUS
─────────────────────────────────────────────────────────────────────────────
Execute: read_page(filter="interactive", depth=10)

Search for button indicators:
  - "Friends" button (for friend accounts)
  - "Following" button (for pages/public figures)
  - "Follow" button (indicates already unfollowed)
  - Three-dot menu "..." (alternative access point)

BUTTON LOCATION LOGIC:
  IF account.button_type == "Friends":
    primary_button = find_button(text="Friends")
    fallback_button = find_button(text="Follow")
  
  ELSE IF account.button_type == "Following":
    primary_button = find_button(text="Following")
    fallback_button = find_button(text="Follow")

IMMEDIATE STATUS CHECK:
  IF fallback_button ("Follow") is found AND primary_button is NOT found:
    # Account is already unfollowed
    RETURN "ALREADY_UNFOLLOWED"

─────────────────────────────────────────────────────────────────────────────
STEP C: OPEN ACTION MENU
─────────────────────────────────────────────────────────────────────────────
IF primary_button NOT found:
  # Button might be hidden or page structure different
  TRY finding three-dot menu "..."
  IF found:
    click(three_dot_menu)
  ELSE:
    RETURN "FAILED"

ELSE:
  # Standard button found - click it
  click_result = click_with_retry(primary_button, max_attempts=3)
  
  IF NOT click_result:
    RETURN "FAILED"

# Wait for menu to appear
Wait 1.5 seconds
Execute: read_page(filter="interactive", depth=8)  # Re-read to get menu

─────────────────────────────────────────────────────────────────────────────
STEP D: LOCATE UNFOLLOW OPTION IN MENU
─────────────────────────────────────────────────────────────────────────────
Search menu for options (case-insensitive):
  - "Unfollow"
  - "Unfollow [account name]"
  - "Following" (submenu that contains unfollow)

DETECTION LOGIC:
  unfollow_option = find_in_menu(["Unfollow", "Unfollow " + account.name])
  follow_option = find_in_menu(["Follow", "Follow " + account.name])
  following_submenu = find_in_menu(["Following"])

IF follow_option found AND unfollow_option NOT found:
  # Already unfollowed (menu shows "Follow" to re-follow)
  Press Escape to close menu
  RETURN "ALREADY_UNFOLLOWED"

IF unfollow_option found:
  # Direct unfollow option available
  PROCEED to STEP E

ELSE IF following_submenu found:
  # Need to open submenu first
  click(following_submenu)
  Wait 1 second
  Execute: read_page(filter="interactive", depth=8)
  unfollow_option = find_in_menu(["Unfollow"])
  IF NOT found:
    Press Escape
    RETURN "FAILED"

ELSE:
  # No unfollow option found at all
  Press Escape
  RETURN "FAILED"

─────────────────────────────────────────────────────────────────────────────
STEP E: EXECUTE UNFOLLOW ACTION
─────────────────────────────────────────────────────────────────────────────
1. Click the unfollow_option element
2. Wait 1 second for confirmation dialog (if any)

Check for confirmation dialog:
  Execute: read_page(filter="interactive", depth=6)
  
  confirmation_button = find_button(["Update", "Confirm", "Unfollow"])
  
  IF confirmation_button found:
    click(confirmation_button)
    Wait 1 second
  
Press Escape key (close any remaining menus/dialogs)
Wait 1 second

─────────────────────────────────────────────────────────────────────────────
STEP F: VERIFY UNFOLLOW SUCCESS
─────────────────────────────────────────────────────────────────────────────
Execute: read_page(filter="interactive", depth=10)

Look for button state change:
  follow_button = find_button(["Follow", "Follow " + account.name])
  friends_button = find_button("Friends")
  following_button = find_button("Following")

VERIFICATION LOGIC:
  IF follow_button found:
    # Successfully unfollowed (button now shows "Follow")
    verification_status = "SUCCESS"
  
  ELSE IF friends_button found OR following_button found:
    # Still showing following status - unfollow may have failed
    verification_status = "FAILED"
  
  ELSE:
    # Ambiguous state - mark for retry
    verification_status = "FAILED"

─────────────────────────────────────────────────────────────────────────────
STEP G: RETURN TO FOLLOWING TAB
─────────────────────────────────────────────────────────────────────────────
Execute: navigate(url="https://www.facebook.com/[USERNAME]/following")
Wait 2 seconds for page load

RETURN verification_status

END FUNCTION process_single_account

═══════════════════════════════════════════════════════════════════════════════
PHASE 3: RETRY & RECOVERY SYSTEM (RECURSIVE)
═══════════════════════════════════════════════════════════════════════════════

RETRY PASS 1: FAILED ACCOUNTS
------------------------------
IF retry_queue is NOT empty:
  
  PRINT "Starting Retry Pass 1: " + retry_queue.length + " accounts"
  
  FOR EACH account IN retry_queue:
    Wait 3 seconds (prevent rate limiting)
    result = process_single_account(account)
    UPDATE account status based on result
  
  END FOR

RETRY PASS 2: STILL FAILED ACCOUNTS
------------------------------------
IF retry_queue is STILL NOT empty:
  
  PRINT "Starting Retry Pass 2 (Final): " + retry_queue.length + " accounts"
  
  FOR EACH account IN retry_queue:
    Wait 5 seconds (longer pause)
    result = process_single_account(account)
    UPDATE account status based on result
  
  END FOR

MANUAL REVIEW LIST
------------------
Any accounts still failed after retry passes → add to manual_review_list

═══════════════════════════════════════════════════════════════════════════════
PHASE 4: COMPREHENSIVE VERIFICATION SWEEP
═══════════════════════════════════════════════════════════════════════════════

PURPOSE: Verify that unfollowed accounts are actually removed from following list

STEP 4.1: RETURN TO FOLLOWING TAB
----------------------------------
Navigate to: https://www.facebook.com/[USERNAME]/following
Press cmd+up (Mac) or ctrl+up (Windows/Linux) to scroll to top

STEP 4.2: RELOAD ENTIRE LIST
-----------------------------
Execute infinite scroll loader (same as Phase 1)
Execute: get_page_text → save as verification_content
Execute: read_page(filter="interactive", depth=12)

STEP 4.3: COUNT REMAINING ACTIONABLE ACCOUNTS
----------------------------------------------
remaining_friends_buttons = count_buttons(text="Friends")
remaining_following_buttons = count_buttons(text="Following")
remaining_actionable = remaining_friends_buttons + remaining_following_buttons

PRINT "Verification Sweep Results:"
PRINT "  Remaining 'Friends' buttons: " + remaining_friends_buttons
PRINT "  Remaining 'Following' buttons: " + remaining_following_buttons
PRINT "  Total actionable remaining: " + remaining_actionable

STEP 4.4: RECURSIVE RE-PROCESSING IF NEEDED
--------------------------------------------
IF remaining_actionable > 0:
  
  PRINT "Found " + remaining_actionable + " accounts still in following list"
  PRINT "Extracting and re-processing..."
  
  # Extract these accounts
  new_queue = extract_actionable_accounts()
  
  # Filter out accounts already in successfully_unfollowed list
  new_queue = filter_already_processed(new_queue)
  
  IF new_queue is NOT empty:
    PRINT "Re-processing " + new_queue.length + " accounts"
    
    # Recursive call: process this new queue
    FOR EACH account IN new_queue:
      result = process_single_account(account)
      UPDATE account status
    END FOR
    
    # Recursive verification: call this sweep again
    GOTO STEP 4.1 (verify again)
  
END IF

STEP 4.5: FINAL CONFIRMATION
-----------------------------
IF remaining_actionable == 0:
  PRINT "✓ Verification complete: No actionable accounts remain"
  PRINT "✓ Only pending friend requests ('Cancel request' buttons) remain"
  PROCEED to Phase 5

═══════════════════════════════════════════════════════════════════════════════
PHASE 5: SPOT-CHECK VERIFICATION (SAMPLE VALIDATION)
═══════════════════════════════════════════════════════════════════════════════

PURPOSE: Random sample of processed accounts to ensure unfollow persisted

SELECT 5 random accounts from successfully_unfollowed list

FOR EACH sample_account:
  
  PRINT "Spot-checking: " + sample_account.name
  
  Navigate to: sample_account.url
  Wait 2 seconds
  Execute: read_page(filter="interactive", depth=10)
  
  follow_button = find_button("Follow")
  
  IF follow_button found:
    PRINT "  ✓ Confirmed: Still unfollowed"
  ELSE:
    PRINT "  ✗ WARNING: May have re-followed or status changed"
    ADD to anomaly_list
  
END FOR

IF anomaly_list is NOT empty:
  PRINT "⚠ Anomalies detected: " + anomaly_list.length + "

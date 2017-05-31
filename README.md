
# Draft for Page/Record Locking in TYPO3

## Purpose &amp; current situation:
TYPO3 implements a "weak advisory record locking". That means whenever a TCE form edits a record it is marked as locked. But this locking is only shown as a warning to the user (the exclamation mark in the pagetree and the yellow field in the pagerecord with the warningtext). Neither the UI nor the API (TCE main) use this locking.

## Goal:

Implement a true locking mechanism where editing of an unlocked record is not possible. The UI has to indicate that a record is locked and the API has to enforce it. Additionally it must be able to unlock a page. Locking a page locks the page record, the alternative language page records and all content records. Depending on the TCA configuration other records than tt_content records may be locked with the page.


Intelligent means  lock managing must be implemented. An admin/user with the right permissons must be able to "break a lock" and after a certain amount of time the locks must expire (escalation handling).

The UI must show the locking state of records in the page tree and the content frame (see Screenshots). A page can only be locked once. A page can be published via a defined workflow (see next chapter) or immediately.

## Changes to the API:


  * TCEmain must get a new command to lock a record. The locking of a page record automatically locks all dependent records (see above).
  * TCEmain must get a new command to unlock a record.
  * TCEmain must not update or delete a locked record.
  * The lock table must be changed to contain the session id.
  * The session clean up must have an option to clear all session locks on exit.


## Changes to the UI:

Two possible modes of operation are possible: _explicit_ and _implicit_ locking. The former requires a user to explicitely lock a record by eg clicking a lock button/icon. The latter locks a record when the user clicks on a edit icon.

## Explicit locking:


  * In this mode the user has by default no edit icons for page related table (if they are shown the must be grayed out and inoperable). In the page tree and in web&gt;page there is a "lock page" option.
  * After locking a page, the UI shows the edit icons for all page related contents of the locked page (thus the complete page + tt_content records). Saving and closing the page does not change the locking state. A page is only unlocked after the user chooses the unlock option in the page tree or web&gt;page.
  * Non page related records (eg. tt_news records) are locked implicitely (see below). It may be possible to define other sets of related records where the locking of a "master record" locks related records (eg a shop where a product has a master records and any number of variant records). In this case web&gt;list may either never show edit icons for the master record or use implicit locking.
  * In this mode it is possible to let a user lock a page even when he navigates around in the page tree etc and closes the record.


## Implicit locking:


  * In this mode the edit icons show up as usual, but by choosing an edit option a record or a page is locked automatically. Choosing "close" unlocks the record again. 
  * In this mode not choosing close leads to a stale lock. The UI has no way to unlock a record besides clicking close.


## Features:


  * Locking-Manager (BE-Module): The admin must be able to list all locks (with user and session info) and to break a lock.
  * Internal messaging system to communicate nearly real-time with other editors (not hidden in taskcenter, directly integrated) =&gt;&gt; this is postponed, move to an extra extension


## Mockups:


  * ![Showing a page which is not locked in explicit mode](/screenshots/v2/1explicit.jpg): 
  * ![Showing a page which ws locked by me](/screenshots/v2/2lockedbyme.jpg):
  * ![Showing a page which was locked by another user](/screenshots/v2/3lockedbyotheruser.jpg):
  * ![BE Modul Features](/screenshots/v2/bemodul.jpg):


## To be discussed:

  * Screens are showing the standard pagemodul, what about templavoila?
  * What about the Listmodul? Recordlocking? (eg. tt_news) 
  * How are   records displayed in the listmodul which are in a workflow process?
  * Howto realise the integration in the listmodule without overwriting too much -&gt; making listmodule its own extension?
  * Howto realise the integration in the pagemodule without overwriting too much -&gt; making pagemodul its own extension?
  * When a page record gets locked, should all alternative language page records also get locked?
  * Messaging System: Masi implemented a new feature in Tools &gt; User Admin which lists all Users online (since 4.1). Could this programming maybe used for the messaging system?


## TODO


  * expiry of locks by time
  * clean-up during log-out
  * BE module for managing locks
  * implicit locking support by other than alt_doc.php
  * optimized "is_locked" function (which currently make lots of expensive look-ups for "extended" mode)


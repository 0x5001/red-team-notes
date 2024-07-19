* Microsoft introduced defenses against download executors since 2016.
* The enterprise groups are in the forest root domain.
* On domain member machines to query local users, you need administrative privileges.
* MemberSID = Group SID + User RID
* Whatever changes we made in ACLs for domain admins which is a member of protected groups group, will be reset every hour.
* Most adversaries mostly purchase initial accesses.
* There is no built-in tool to catch reverse-shells in Windows OS.
* PowerCat is a reverse-shell script written in PowerShell.
* `$ExecutionContext.SessionState.LanguageMode` -> To check Language Mode. (Constrained Language mode is active or not.)
	* In Constrained Language mode, we can just only execute built-in cmdlets.
* Domain DPAPI backup keys, can be used to decrypt DPAPI protected credentials for any user. There is no way to rotate the domain DPAPI backup key. (You can extract from lsass process or ntds.dit file)
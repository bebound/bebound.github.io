+++
title = "Internet Account Keeps Coming Back after deletion on MacOS"
author = ["KK"]
date = 2022-01-08T00:09:00+08:00
lastmod = 2023-12-18T21:38:38+08:00
tags = ["MacOS"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

Today I tried to delete an inactive Internet account on system preference. It was deleted successfully but come back again after 20 seconds. This drives me nuts.

I tried these methods, but none of them works.

-   Boot in [safe mode](https://support.apple.com/en-us/HT201262), delete account.
-   Delete record in `ZACCOUNT` table in `~/Library/Accounts/Accounts4.sqlite`.
-   Delete related items in Keychain Access app.

Later, `RedHatDude`'s answer in [this post](https://discussions.apple.com/thread/252924363?login=true) gives me a clue, it looks like a iCloud sync problem. I tried to delete the account on my 3 MacBooks together. Thank goodness! It does not show up again.


## Ref {#ref}

1.  [Removing Accounts from Internet Accounts](https://community.jamf.com/t5/jamf-pro/removing-accounts-from-internet-accounts/td-p/179654)
2.  [Every time I delete an internet account it comes back](https://www.twit.community/t/every-time-i-delete-an-internet-account-it-comes-back/9914)
3.  [Gmail account keeps coming back after delete](https://discussions.apple.com/thread/252924363?login=true)

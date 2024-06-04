# Setting up Windows samba share (when your unraid shares are public)

Windows might ask you to "Enter network credentials" even when you try to add your public folders from unraid into your machine. Since these folders are not password protected, this is more a quirk of windows. Use the following to add

1. Use 'guest' as the username
2. Use '1234' as the password
3. Tick 'remember my credentials'

This should work and windows should not ask you for the credentials again, but if it does, go to the windows credential manager and create/save those credentials.
Voicemail databases cleanups

This code will:

- first ensure that all databases listed as `user_database` in local numbers records actually exist and have a non-empty securty document; otherwise, re-save the local-number record to force creation;
- conversely, list any database that has a `voicemail_settings` field but is not listed as a `user_database`;
- ensure all databases are compacted;
- remove outdated voicemail messages;
- cleanup voicemail database if the local-number has been disabled for > 45 days.

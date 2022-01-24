# Migrate E-mail

| **IMPORTANT** | Follow this tutorial at your own risk!<br/>I will not be held accountable for any lost E-mail etc. |
| ------------- | :--------------------------------------------------------------------------------------------- |

I've written this short tutorial for myself, in order to replicate my effort of backing up, and restoring my G-Suite mailboxes to another IMAP provider.
It worked for me, it might not work for you.
Even though it did not eat my cookies, steal my bike, and kill my cat, it might just do it to you.

_Theoretically speaking_, you should be able to restart the entire process as long as you don't accidentally run `imap-backup restore` or `IMAPdedup` on the source IMAP server.
You will at one point have to change the configuration from the source IMAP server to the target IMAP server, take extra care at that point and triple check your configuration!

## Create back-up

Download [`imap-backup`](https://github.com/joeyates/imap-backup) to create a back-up from the source IMAP server.
Configure all settings using `imap-backup setup` and simply run `imap-backup` afterwards to start the download.
This will create corresponding directory structure in `~/.imap-backup` with the E-mail adress (e.g., `bob.debouwer_bouwerbv.com/`).
This process will take some time, depending on the amount of data being transfered.

Every mail directory (e.g., `Sent Mail`) will have its own `.mbox` and `.imap` file.
The `.mbox` file will have all the actual E-mail content in it, the `.imap` file will have all the `UIDs` for these E-mail.

On oddity when you're downloading from Google is that there will be an `All Mail.mbox` and corresponding `.imap` file, which will also contain a copy of _every single E-mail_ in all other `.mbox` files.
On Google's side only one copy exists in `All Mail.mbox` and through magic divided/shown over multiple directories, for the `IMAP` protocol however they become seperate E-mail.
This means you'll have to deduplicate them at the end...

## Restore back-up to new server

By default `imap-backup` will place some `.mbox` and `.imap` files in subdirectories.
It does not however, support restoring them in this fashion `¯\_(ツ)_/¯`.
You'll have to move every file pair in to the main directory for that E-mail address.

After the back-up is complete, you can edit the configuration using `imap-backup setup` to the new server settings or edit `~/.imap-backup/config.json` directory.
Directly afterwards you can run `imap-backup restore` to upload all E-mail up to the new server.
This process might take a while, but when it's done all your E-mail should be on the target IMAP server in their original IMAP directory.

## Deduplication

At this point you'll have duplicates of a lot of E-mail in an IMAP directory called `All Mail` which you'll now need to deduplicate.
To resolve this, you can use a tool called [`IMAPdedup`](https://github.com/quentinsf/IMAPdedup), this will deduplicate all IMAP directories in the order which is given parameters.

First, you'll need a list of all the IMAP directories on the target server and export it to a file:

```SHELL
./imapdedup.py --server <TARGET IMAP SERVER> --port 993 --ssl --user "<USERNAME>" --password <PASSWORD> --verbose --list > directories
```

Edit the `directories` file and order the directories in which to deduplicate, where any E-mail found in earlier directories will be deduplicated from later directories.
At the same time, place `"` around all directories containing spaces (' ') (e.g., `All mail` becomes `"All mail"`).
An example file will probably have `INBOX`, and `"Sent Mail"` at the top, and `"All Mail"` at the bottom:

```SHELL
# Example file
INBOX
"Sent Mail"
Sent
Drafts
Important
Scheduled
LinkedIn
DMARC
CNCF
Chats
Starred
Trash
INBOX.spam
Spam
"All Mail"
```

In this example all duplicates should be removed from `"All Mail"`.

You can now run:

```SHELL
cat directories | xargs ./imapdedup.py --server <TARGET IMAP SERVER> --port 993 --ssl --user "<USERNAME>" --password <PASSWORD> --dry-run
```

This should tell you what `IMAPdedup` is about to do once you remove the `--dry-run` parameter.
If you're confident everything is correct, proceed in doing so.

At the end you should have a set of directories on your new IMAP server without duplicates.

## Optional administrative tasks

Depending on your preferences and target server, you can do one or more of the following tasks manually in your mail client.

- Move mail from `Sent Mail` into `Sent`
- Rename `All Mail` to `Archive` and set the directory as target for archived E-mail
- Add any filters you had on your previous IMAP server

## Change your MX records

When all is set and done, you'll need to change your MX records.
This is outside of the scope of this HOWTO, since it depends on a lot of things.
Just keep in mind that many mail clients allow you easily move a couple of E-mail between servers by dragging them.
You can use that to move the last few straggler E-mail.

## tl;dr

- Download [`imap-backup`](https://github.com/joeyates/imap-backup), run setup (`imap-backup setup`), and download your mail (`imap-backup`)
- Edit `~/.imap-backup/config.json` and replace source IMAP server with target IMAP server
- Run `imap-backup restore`
- Download and run [`IMAPdedup`](https://github.com/quentinsf/IMAPdedup) to deduplicate all the E-mail
- Move some mail around on your new server
- Don't for get to change your MX records

## Thanks to

- [Joe Yates](https://github.com/joeyates) - For creating [`imap-backup`](https://github.com/joeyates/imap-backup)
- [Quentin Stafford-Fraser](https://github.com/quentinsf) - For creating [`IMAPdedup`](https://github.com/quentinsf/IMAPdedup)

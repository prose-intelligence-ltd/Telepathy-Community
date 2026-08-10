## Telepathy: An OSINT toolkit for investigating Telegram chats. Developed by Jordan Wildon. Version 2.3.4.

Telepathy has been described as the "swiss army knife of Telegram tools," allowing OSINT analysts, researchers and digital investigators to archive Telegram chats (including replies, media content, comments and reactions), gather memberlists, lookup users by given location, analyze top posters in a chat, map forwarded messages, and more.

The toolkit has already seen a wide variety of use cases, including but not limited to: in investigative and data journalism, by academic and research institutions, and for intelligence gathering and analysis.



## Purpose

Telepathy-Community is the public release of Telepathy, a Python command-line OSINT toolkit for collecting and analysing public Telegram chats. `setup.py` installs a single console entry point, `telepathy`, bound to `telepathy.telepathy:cli`; the toolkit's features (basic and comprehensive chat scans, memberlist collection, forward edgelists, media archiving, user and location lookups, chat export, reply retrieval and translation) are described in the usage sections below. The repository is public, MIT-licensed, and carries 1,232 stars and 161 forks. It has no internal runtime and nothing in the Prose estate depends on it: `prose-catalogue` records `runtime: none`. External users are the only consumers of this repository.

**This repository is not actively maintained, and the rest of this README does not reflect that.** The last change to the toolkit's own source was on 2024-07-12. Every commit since is repository housekeeping: security-workflow caller stubs on 2026-08-07 and a Dependabot dependency bump on 2026-08-10. The published package on PyPI, `telepathy` 2.3.4, was last uploaded on 2024-07-12 and has not been republished since. There are 41 open issues; installation and runtime failure reports filed between 2023 and 2025 stand with no maintainer reply. Sections of this README written in the future tense — the upcoming-changes and upcoming-features lists, and the statements that deeper analytics and further location-scanning support are planned or being explored — describe work that is not in progress. `src/telepathy/const.py` likewise still sets `__status__ = "Development"`.

Two concrete consequences for anyone arriving from the installation instructions:

- **The install-from-source path is broken on the default branch.** `src/telepathy/telepathy.py` does not parse: `class PlaceholderClass:` at line 1743 is followed by an unindented body, raising `IndentationError` at line 1744. The package's main module is therefore unimportable, and `pip install -r requirements.txt` against a fresh clone produces a non-working `telepathy` command. The stale duplicate under `build/lib/telepathy/` does parse, but it is a 2024 copy that is not what an install uses. The package directories also contain `__init.py__` rather than `__init__.py`, in both `src/` and `src/telepathy/`.
- **The two documented install paths do not agree.** `setup.py` requires `telethon == 1.36.0` while `requirements.txt` pins `Telethon==1.25.2`, so a pip install and a source install get different core libraries. Version strings disagree too: this README and `setup.py` say 2.3.4, `src/telepathy/const.py` says 2.3.2.

The `pip3 install telepathy` path installs the July 2024 release, which predates none of the open bug reports above and has not changed since. UNKNOWN: whether the published 2.3.4 sdist carries the same syntax error as the repository source — the sdists committed under `dist/` are 2.3.2 and earlier, and the published 2.3.4 artefact was not downloaded or inspected. The clone URL given in the installation section points at the repository's former path and resolves to the current location via a GitHub redirect (HTTP 301), so it still works.

This section is written against the default branch, `main`, at commit `f1a5b95`.

## Ownership

Owner: **Al Baker — al@prose.ltd**, sole operator. This is confirmed by `prose-catalogue` (`owner: al`) and by the `owner-al` topic on the GitHub repository. There is no `CODEOWNERS` file on the default branch.

Escalation for anything concerning this repository goes to al@prose.ltd. External bug reports are not currently triaged, and no response time is offered — the open-issue record above is the accurate expectation to set.

The contact routes named elsewhere in this README, in the package metadata (`setup.py` `author_email`, and `__maintainer__` and `__email__` in `src/telepathy/const.py`) and in the CLI start-up banner printed by `src/telepathy/utils.py` are a former maintainer's personal address and social account. They are historical attribution, not Prose Intelligence support channels, and correspondence sent to them will not reach the owner.

`prose-catalogue` records `lifecycle: development` for this repository, but the GitHub repository carries no `lifecycle-*` topic — its only topic is `owner-al`. That divergence is documented here rather than resolved: changing a repository's lifecycle alters which CI checks are required, and is the owner's decision alone.
## Are you looking for a enterprise-grade version of Telepathy?
Visit [prose.ltd](https://prose.ltd) to find out how we can turbocharge your Telegram data collection with Telepathy Pro. No accounts, dealing with the command line, or hassle needed!



## Installation

### Pip install (recommended)

```
$ pip3 install telepathy
```

### Install from source

```
$ git clone https://github.com/jordanwildon/Telepathy.git
$ cd Telepathy
$ pip install -r requirements.txt
```

## Setup

On first use, Telepathy will ask for your Telegram API details (obtained from my.telegram.org). Once those are set up, it will prompt you to enter your phone number again and then send an authorization code to your Telegram account. If you have two-factor authentication enabled, you'll be asked to input your Telegram password.

OPTIONAL: Installing cryptg ($ pip3 install cryptg) may improve Telepathy's speed. The package hand decryption by Python over to C, making media downloads in particular quicker and more efficient. 


## Usage:

```
telepathy [OPTIONS]
```

Options:
- **'--target', '-t' [CHAT]**

this option will identify the target of the scan. The specified chat must be public or have a private link. To get the chat name, look for the 't.me/chatname' link, and subtract the 't.me/'.

For example:

```
$ telepathy -t durov
```

The default is a basic scan which will find the title, description, number of participants, username, URL, chat type, chat ID, access hash, first post date and any applicable restrictions to the chat. For group chats, Telepathy will also generate a memberlist (up to 5,000 members).


- **'--comprehensive', '-c'**

A comprehensive scan will offer the same information as the basic scan, but will also archive a chat's message history, gather the number of reactions, archive how many times a message has been forwarded, the number of replies to each message, and more.

Reaction lists are included in the archive file, including basic calculations of engagement rate. Only the most-common reactions are listed, with the total including all possible reactions. Currently, Telepathy calculates engagement rates based on forwards, comments and reactions seperately, with a calculation based on post views and one based on chat participant count. In future, Telepathy may include deeper analytics which can be cross-compared between chats based on a combination of these metrics, fixing for when comments, reactions or forwards are allowed or disallowed in a given chat.

For example:

```
$ telepathy -t durov -c
```


- **'--forwards', '-f'**

This flag will create an edgelist based on messages forwarded into a chat. It can be used alongside either a default or comprehensive scan. Since 2.3.0, Telepathy now formats these edgelists to maximize compatability with Gephi.

For example:

```
$ telepathy -t durov -f

$ telepathy -t durov -c -f
```


- **'--media', '-m'**

Use this flag to include media archiving alongside a comprehensive scan. This makes the process take significantly longer and should also be used with caution: you'll download all media content from the target chat, and it's up to you to not store illegal files on your system.

To archive media, you must run a comprehensive scan:

```
$ telepathy -t durov -c -m
```

Once files have downloaded, you can run exiftool on the associated media directory to gather deeper insights on the files, their metadata, and in some cases attribute who might be behind an anonymous channel. Further details are in the "bonus investigations tips" section of this README.


- **'--user', '-u'**

Looks up a specified user. This will only work if your account has "encountered" the user before (for example, after archiving a group), you can specify User ID or @nickname. If looking up by username, it's not always necessary for your account to have already seen the user.

```
$ telepathy -t 0123456789 -u

$ telepathy -t @test_user -u
```


- **'--location', '-l']**

Finds users near to specified coordinates. Input should be longitude followed by latitude, seperated by a comma. This feature only works if your Telegram account has a profile image which is set to be publicly viewable. As of 2.3.4, this feature now includes channel lookups. 

While searches for multiple locations at once may work in some cases, Telegram appears to have a limit on how quickly an account can cycle through locations. At the time of writing, this appears to be at least ten minutes. Further location scanning support while using multiple accounts is being explored for a future release.

```
$ telepathy -t 51.5032973,-0.1217424 -l
```


- **'--alt', '-a' [NUMBER]**

Flag for running Telepathy from an alternative number or API details. You can use the same API key and Hash but authenticate with a different phone number. This allows for running multiple scans at the same time. Telepathy will default to the first details you offer, and up to four others can be added. Please see the notes at the top of this README for information regarding limitations with user IDs using this method.

```
$ telepathy -t Durov -c -a 1
```


- **'--export', '-e'**

Exports all chats your account is part of to a CSV file. In a future release, this may assist with provisioning new accounts to automatically following the listed groups.

```
$ telepathy -e
```
  

- **'--reply', '-r'**

Flag for enabling channel reply retrieval, this will archive replies and list users who replied to messages in the target channel. 

```
$ telepathy -t [CHANNEL] -c -r 
```


- **'--translate', '-tr'**

Flag for enabling auotmatic translation (currently only into English) during message retrieval.

```
$ telepathy -t [CHANNEL] -c -tr
```


## Bonus investigations tips:

 - Navigating to a media archive directory and running Exiftool may give you a whole host of useful information for further investigation. Telegram doesn't currently scrub metadata from PDF, DOCX, XLSX, MP4, MOV and some other filetypes, which offer creation and edit time metadata, often timezones, sometimes authors, and general technical information about the perosn or people who created a media file.  
 ```
$ cd ./telepathy/telepathy_files/CHATNAME/media
$ exiftool * > metadata.txt
```
 - Group and inferred channel memberlists offer a point of further investigation for usernames found. By using [Maigret](https://github.com/soxoj/maigret), you can look up where else a username has been used online. While this is not accurate in all cases, it's been proven to be helpful for identifying where a person has reused handles across platforms. In this case, remember to verify your findings to avoid false positives.


## A note on how Telegram works

Telegram chats are organised into three key types: Channels, Megagroups/Supergroups and Gigagroups. Each option works slightly differently depending on the chat type. Channels can have seemingly unlimited subscribers and are where an admin will broadcast messages to an audience, Megagroups can have up to 200,000 members, each of whom can participate (if not restricted), and Gigagroups sit somewhere between the two.


## Upcoming changes
In some environments (particularly Windows), Telepathy struggles to effectively manage files and can sometimes produce errors. Fixes for these errors will come in due course.

Upcoming features include:

  - [ ] Adding a time specification flag to set archiving for specific period.
  - [ ] A new method to once again gather complete memberlists (currently restricted by the API).
  - [ ] Ensuring inferred channel memberlists don't contain duplicate entries.
  - [ ] Exploration of whether channel events can be included, such as name changes.

## feedback

Please send feedback to @jordanwildon on Twitter. You can follow Telepathy updates at @proseltd.


## Usage terms

You may use Telepathy however you like, but your usecase is your responsibility. Be safe and respectful. If your usecase is for commercial purposes, please consider cour enterprise options at [prose.ltd](https://prose.ltd)


## Credits

All tools created by Jordan Wildon (@jordanwildon). Special thanks go to [Giacomo Giallombardo](https://github.com/aaarghhh) for adding additional features and code refactoring, [jkctech](https://github.com/jkctech/Telegram-Trilateration) for collaboration on location lookup via the 'People Near Me' feature, and Alex Newhouse (@AlexBNewhouse) for his help with Telepathy v1. Shoutout also to [Francesco Poldi](https://github.com/pielco11) for being a sounding board and offering help and advice when it comes to bug fixes.

Where possible, credit for the use of this tool in published research is desired, but not required. This can either come in the form of crediting Jordan Wildon, Prose Intelligence, or crediting Telepathy itself.

---
title: "Taking Notes With Nextcloud Qownnotes and Notebooks"
date: 2017-05-17T15:49:10-06:00
draft: false
type: "post"
tags:
  - notes
  - nextcloud
  - linux
  - qownnotes
  - ios
  - php
aliases:
  - /blog/taking-notes-with-nextcloud-qownnotes-and-notebooks
---

I've seen a lot of people on Twitter recently asking for advice on taking notes. Mostly as alternative to Evernote. I've shared my setup when I can, but 140 characters only goes so far, so I thought I'd write up something more in-depth. Personally, I like the ability to jot down something on-the-go and then do something with it later. That means I need a way to sync file from my phone to my PC. My setup has three key pieces: [Nextcloud](https://nextcloud.com/), [QOwnNotes](http://www.qownnotes.org/), and [Notebooks for iOS](http://www.notebooksapp.com/).

## Nextcloud

I use Nextcloud for backup and syncing files. This workflow revolves around a functioning Nextcloud (or ownCloud) server. Setup for Nextcloud/ownCloud will very slightly depending on your server, but the [official documentation](https://docs.nextcloud.com/server/11/admin_manual/installation/index.html) is a great place to start. I'm hoping to document my setup here soon.

## Notes (Nextcloud app)

There is an official Notes app for Nextcloud that other applications integrate with. Setting it up is pretty painless. Simply log into your Nextcloud server and go to **Apps**.

![Screenshot of the Nextcloud app menu](/images/nextcloud-01.png 'Screenshot of the Nextcloud app menu')

Go to the **Office & text** section.

![Partial screenshot of Nextcloud App categories menu](/images/nextcloud-02.png 'Partial screenshot of Nextcloud App categories menu')

Find the **Notes** app and click the Enable button. This will download and enable the app.

![Screenshot of the Notes app entry](/images/nextcloud-03.png 'Screenshot of the Notes app entry')

After it's enabled, you will find a Notes icon under the Apps menu.

![Screenshot of the Nextcloud app menu with the Notes app listed](/images/nextcloud-04.png 'Screenshot of the Nextcloud app menu with the Notes app listed')

The first visit to the Notes app will create a Notes folder in the root of your Nextcloud files.

## QOwnNotes

QOwnNotes is an excellent, cross-platform note-taking app that (when paired with Nextcloud) let's you control your data. Learn more and follow the instructions for your OS at the [QOwnNotes website](http://www.qownnotes.org/installation).

## Setup QOwnNotes + Notes integration

While you can use QOwnNotes without a Nextcloud backend, the integrations allows you to sync your notes to multiple devices and access previous versions and detected notes.

When you first start QOwnNotes, it will ask you to choose where to store your notes. If you already have Nextcloud sync tool running, the Notes folder that was created when you visited the Notes app in Nextcloud will already be on your computer. If not, you will need to create a folder to hold your notes. Browse to select the folder.

![Screenshot of the first screen of QOwnNotes setup](/images/qownnotes-01.png 'Screenshot of the first screen of QOwnNotes setup')

I tend to skip syncing initially and get the program up and running first. Click **Next** to skip those settings.

![Screenshot of the second screen of QOwnNotes setup](/images/qownnotes-02.png 'Screenshot of the second screen of QOwnNotes setup')

**Finish** the setup.

![Screenshot of the final screen of QOwnNotes setup](/images/qownnotes-03.png 'Screenshot of the final screen of QOwnNotes setup')

Notebooks for iOS uses "books" to organize which correlates with note subfolders in QOwnNotes. Go to Note > Settings > Note folders. Choose "Use note sub-folders." You will now see a list of subfolders above the list of notes. I had restart the program to see the full list.

![Screenshot of the Note Folders section of QOwnNotes settings dialog](/images/qownnotes-04.png 'Screenshot of the Note Folders section of QOwnNotes settings dialog')

Now set up the Nextcloud sync. Go to Note > Settings > ownCloud / Nextcloud.

![Screenshot of the ownCloud/Nextcloud section of QOwnNotes settings dialog](/images/qownnotes-04.png 'Screenshot of the ownCloud/Nextcloud section of QOwnNotes settings dialog')

Enter your URL and credentials. I suggest setting up [two-factor authentication](https://docs.nextcloud.com/server/11/user_manual/user_2fa.html#using-client-applications-with-two-factor-authentication) and [device passwords](https://docs.nextcloud.com/server/11/user_manual/session_management.html) in your Nextcloud settings. Any existing notes will now be synced.

## Setup Notebooks

Notebooks does have a PC and Mac version, but nothing native for Linux which is why I went with QOwnNotes. I use both an iPhone and iPad so Notebooks works well for me on those platforms. Vist the [App Store](http://itunes.apple.com/app/id780438662?ls=1&mt=8) to purchase and download. There is also a slightly cheaper, [iPhone-only version](https://itunes.apple.com/us/app/notebooks-for-iphone/id780442075?mt=8). Pick whichever suits your needs.

Open Notebooks

![Screenshot of empty Notebooks app](/images/notebooks-01.png 'Screenshot of empty Notebooks app')

Go to Settings > Import, Export, Sync

![Screenshot of Notebooks app settings](/images/notebooks-02.png 'Screenshot of Notebooks app settings')

Turn on WebDAV and enter Settings

![Screenshot of Notebooks app Import, Export, Sync settings](/images/notebooks-03.png 'Screenshot of Notebooks app Import, Export, Sync settings')

Enter the WebDAV settings for your Nextcloud server and Save

![Screenshot of Notebooks app WebDAV settings](/images/notebooks-04.png 'Screenshot of Notebooks app WebDAV settings')

Press Done and Done (again) to get back out of Settings.

Touch the Sync icon (your WebDav server should be showing)

![Screenshot of Notebooks app sync popup](/images/notebooks-04.png 'Screenshot of Notebooks app sync popup')

Now you're setup for on-the-go note taking. One caveat: as of the time of this post, WebDav sync was not automatic. So after creating/updating notes in Notebooks, remember to manualy resync.

### Notebooks tweaks

Since QOwnNotes works with Markdown files, I make a couple of tweaks to Notebooks' settings to ensure compatibility

* Settings > Markdown Settings > Enable List Markdown as Extra Document Type
* Settings > Write & Edit > Default Document Type > Markdown
  * Markdown will only be an option if you enable it as an extra document type

## CloudNotes (another option)

I've used [CloudNotes](http://peterandlinda.com/cloudnotes/) for accessing my notes on my phone since I first started using Nextcloud/ownCloud and the Notes app. After finding QOwnNotes, I wanted something with Markdown support and found Notebooks. Shortly after I started using Notebooks, CloudNotes announced Markdown support. If you don't have the need for sub-notebooks, CloudNotes may be an better option for you since it has direct integration with the Notes app instead of using WebDav.

Happy Note-taking!

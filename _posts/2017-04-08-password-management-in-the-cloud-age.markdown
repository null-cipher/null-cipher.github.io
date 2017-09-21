---
layout: post
title:  "Password Management in the Cloud Age"
date:   2017-04-08 08:37:01 +1200
---

As has been highlighted by better writers than I, we have a problem with [the way that we think about passwords][xkcd-passwords].
Really it boils down to the question "From whom am I protecting this password?"
In the early days of computing, you were protecting it from your co-workers or other people that had physical access to your computer.
As such, writing your password on a post-it note stuck to your monitor was a phenomenally bad idea as both of the means of access were in the same place.
Now that most of our computers are connected to the internet it is not people that we have to protect our passwords from, but rather from computers.
To this end, even Schneier has suggested [writing down your passwords][schneier-passwords] on a card you keep in your wallet because the biggest threat to you probably isn't even in the same country.


# Making better passwords
The types of passwords that are easy for people to remember tend to be easy for computers to guess.
Humans are unique in their capacity for recognising patterns, but computers are also quite good at this now.
An excellent demonstration of computers being good at recognising patterns in passwords is shown in the [Passfault][passfault] project from OWASP.
The hardest password to guess is one that is composed entirely of random characters, as this will have the highest possible entropy and be harder to guess.
But being hard to guess will make it nearly impossible to remember.
How can we overcome this limitation?

# Password managers
In following with best security practices, you should [never reuse passwords][pwd-reuse].
If you need to create strong, random passwords for the many accounts, services and websites we use every day you would have no hope of remembering them and having to write them all down would be unwieldy.
In recent years, password managers have become popular among end-users as well as enterprises (which have always required methods for storing and sharing credentials).
These systems have been an excellent step forward in security but have chiefly suffered from the inability for their password database to be portable.
Not being able to log into your Facebook account on your smartphone because you can't remember your 15 character random password and can't reach your desktop to fetch it, is not ideal.

In the past few years, cloud-based password managers such as LastPass have gained a lot of popularity for the fact that you can access your passwords anywhere, at any time.
However, LastPass has been [plagued][lphack-2015] [with][lphack-2016] [security][lphack-2017] issues and requires a great deal of trust in a 3rd party to protect your data.
There is a better way to protect your passwords, while still benefiting from the portability and convenience of the cloud.


# Keepass 2 architecture
The high-level overview of the system we are going to build is as follows:
1. We are going to install [Keepass 2][keepass] on our main computer. Keepass is available Windows/Linux/OS X, but a specific installation guide is beyond the scope of this article. You can download it [here][keepass-download]. Be sure to download the 2.xx version, not the "Classic Edition".
2. We are going to initialise the password database to use a master password and a key file
3. We will install a cloud sync plugin for Keepass to allow it to copy the encrypted password database to a cloud storage location ([GoogleSyncPlugin][gsync] in our case)
4. We will then save the database and trigger a sync to copy it to our external location

There are a number of benefits to doing things this way. Chiefly, you don't have to share your secrets with your cloud provider.
Your password database is encrypted by the time it reaches them.
In order to appropriately protect your passwords you should never copy your key file to your cloud provider.
With the database and the key in their possession, they need only guess your master password to gain access to all of your secrets.


# Keepass Setup
After installing the Keepass application to your computer you will need to create the password database and the supporting credentials from the File > New... toolbar menu.
This will bring you to the "Create Composite Master Key" wizard where we will provide a strong master password and generate a key.

<figure>
  <img src="{{ site.baseurl }}/assets/keepass02-masterpwd.png" alt="Create composite master key screenshot">
  <figcaption>
    The "Create Composite Master Key" dialog
  </figcaption>
</figure>

Note: Your master password does not need to be a randomly generated one, but it will need to be strong.
Make it as long as you can reliably remember as it will be the main thing protecting *all* of your online accounts.

The interesting thing about Keepass is that you can use any file as a key, including documents or JPEGs.
Ideally, you want your key file to be as random as possible but there may be some value in disguising your key as a random spreadsheet hidden in your files.

You can also include the "Windows user account" in your key, but you will then only be able to access the database from this computer (or in the current domain, for an Active Directory enabled network).
We won't be doing this as we want the database to be availble on multiple different devices.

Click "Create" under the key option and save it to a file on your current computer.
Note that you should ensure that this does not get saved to a network share or a cloud-mapped file that may be shared to other users.
Follow the Entropy Collection steps to generate a strong key.

<figure>
  <img src="{{ site.baseurl }}/assets/keepass03-keygen.png" alt="Key generation tool screenshot">
  <figcaption>
    Key generation in Keepass
  </figcaption>
</figure>

In Step 2 you can just set a name for the database as the defaults are very sensible for protecting data.
Feel free to make adjustments to the settings if you feel they are appropriate.

<figure>
  <img src="{{ site.baseurl }}/assets/keepass04-dbsettings.png" alt="Database settings screenshot">
  <figcaption>
    Key generation in Keepass
  </figcaption>
</figure>

You should now have an active database that comes pre-populated with a handful of example credential entries.

<figure>
  <img src="{{ site.baseurl }}/assets/keepass05-created.png" alt="Database created screenshot">
  <figcaption>
    The pregenerated database
  </figcaption>
</figure>

You can find a quick outline of how to use Keepass before setting up syncing to the cloud in the [KeePass Help Center][keepass-help]


## Google Drive Sync Setup
The setup for mirroring our passwords into Google Drive is remarkably easy. Simply [download][gsync] the GoogleSyncPlugin for Keepass, extract it and copy the GoogleSyncPlugin.plgx to the C:\Program Files (x86)\KeePass Password Safe 2\Plugins\ folder.
Restart the application and you should see it listed under "Tools > Google Sync Plugin".
You will need to add a Credential entry under "General" with a URL of <https://accounts.google.com>
If the entry does not get listed in the plugin configuration you can manually add the String field "GoogleSync.ActiveAccount" to it.

<figure>
  <img src="{{ site.baseurl }}/assets/keepass06-stringfield.png" alt="Google Sync Plugin string field setting screenshot">
  <figcaption>
    Changing the String field setting for the Google Sync Plugin credential
  </figcaption>
</figure>

From the configuration item under that menu entry you can add the credential you just created as the primary identifier.
If you go to Tools > Google Sync Plugin > Upload to Google Drive it will prompt you for your Google Account credentials and to allow access to your drive by the plugin.
Now, whenever you make a change to your local password database it will be copied to Google Drive.

## Key distribution and hot tips
You must ensure that you keep your key safe.
If you are using a key that was generated by Keepass it may be worth printing off a copy of it an storing it with your other protected documents.
*If you lose your key, you lose access to your passwords*

For this reason, I do not have my email password generated by Keepass.
I use a strong, memorable password such that if I were to lose access to my password database I could still access my email account and perform password resets.

If you intend to use your password database on portable devices, you will need to manually copy your key to it.
You cannot use a cloud sharing service to distribute the key to the devices or you risk third parties being able to intercept it and decrypt your password database.
You should also consider the risk of attaching your portable device to untrusted computers if you store your key in it's internal storage.

# Bonus section: Android Setup
In order to access your passwords from an Android device you will need to install one of the Keepass compatible Android apps.
Personally, I use [Keepass2Android Password Safe][android-app] which has performed well for me in the past.
When opening a database you can specify a Google Drive location out-of-the-box and when unlocking it, it will prompt you for a password and a key file.
You can copy the key file you generated from your desktop to your Android device.
You should now be able to view your passwords on your smartphone!

[xkcd-passwords]: https://xkcd.com/936/
[passfault]: http://www.passfault.com/
[schneier-passwords]: https://www.schneier.com/blog/archives/2005/06/write_down_your.html
[pwd-reuse]: http://www.pcworld.com/article/219303/password_use_very_common_research_shows.html
[lastpass]: https://www.lastpass.com/
[lphack-2015]: https://blog.lastpass.com/2015/06/lastpass-security-notice.html/
[lphack-2016]: https://www.theregister.co.uk/2016/07/27/zero_day_hole_can_pwn_millions_of_lastpass_users_who_visit_a_site/
[lphack-2017]: https://www.bleepingcomputer.com/news/security/lastpass-bugs-allow-malicious-websites-to-steal-passwords/
[keepass]: http://keepass.info/
[keepassdownload]: http://keepass.info/download.html
[gsync]: http://keepass.info/plugins.html#kpgsync
[android-app]: https://play.google.com/store/apps/details?id=keepass2android.keepass2android
[keepass-help]: http://keepass.info/help/base/firststeps.html

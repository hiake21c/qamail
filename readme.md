QA Mail
=============
QA Mail lets you create unlimited number of disposable mailboxes and read letters in a cozy web-interface. It is useful for testing web applications that deal with e-mail.
QA Mail can be used both for manual testing via a web user interface and also for automated testing, using its REST API.

To Do List
------------

* Asynchronous notifications about new messages
* Message forwarding to custom addresses
* Use proper HTTP return codes for error handling in API

Preparing for installation
------------
* You should bind a domain name to the IP address of your server.  

Installation via Docker
------------
The easiest way to run QA Mail is to use docker-compose. To do so follow these steps:
* QA Mail docker set up relies on a dedicated SMTP server. Most Linux distros come with a preconfigured SMTP server. Therefore, before running QA Mail, you must stop or uninstall your existing mail server. If you are using debian, this command will be sufficient:
```
systemctl stop exim4 && systemctl disable exim4
```
* Create a file named ".env" in the directory you placed the contents of this repository. Define the hostname variable in this file:
```
HOSTNAME=PLACE_YOUR_DOMAIN_NAME_HERE
```
* Build docker images. This may take a while
```
docker-compose build
```
* Start the application
```
docker-compose up -d
```
* Try to open QA Mail in browser to check everything works fine.  

Manual installation
------------
* Configure the mail delivery agent.  
QA mail needs an external mail delivery agent (MDA). The MDA must be configured so that messages to any address on a specific domain will go into one maildir. Than you should point QA Mail to this Maildir by specifying it in settings.yml.
If you wish yo use Postfix, its configuration is described in the section "How to configure Postfix to work with QA Mail".
* Install postgresql. Create a user, give this user privilage to create databases.
* Fill in settings.yml.example and rename it to settings.yml.
*  Install [rvm](https://rvm.io/rvm/install) (if you don't already have it)
*  Install fresh ruby
```
rvm install 2.3.8
rvm --default use 2.3.8
```
*  Install libraries
```
bundle install
```
* Create the DB
```
rake db:create
rake db:schema:load
rake db:migrate
```
* Start the importer:
```
ruby letter_import.rb
```
* Start the web interface:
```
bundle exec puma -e production -d
```


How to configure Postfix to work with QA Mail
------------

This manual will help you configure Postfix to work with QA Mail on Debian 7 (should also work for Ubuntu and other debian-based Linux distros).

* Install Postfix
```
apt-get install postfix postfix-pcre
```
In the dialog choose "Internet site".
Enter your domain.

* Configure Postfix
```
postconf -e "home_mailbox = Maildir/"
postconf -e "mailbox_command = "
postconf -e "myhostname = YOUR_DOMAIN_NAME"
postconf -e "myorigin = YOUR_DOMAIN_NAME"
postconf -e "mydestination = localhost.localdomain, localhost, YOUR_DOMAIN_NAME"
postconf -e "relay_domains = YOUR_DOMAIN_NAME"
postconf -e "virtual_alias_maps = pcre:/etc/postfix/wildcard.pcre"
```

* Configure address rewriting
Put the following into the file /etc/postfix/wildcard.pcre:
(This is for domain name example.example.com, change it for your domain name.)
```
/^\w+\@example\.example\.com$/ qamail
```

* Restart Postfix
```
/etc/init.d/postfix restart
```

* Add a user, that will run QA Mail
```
adduser qamail
```
Now configure QA Mail. The Maildir, which you should specify in settings.yml is /home/qamail/Maildir/
QA Mail itself must be run by user qamail.
Also, make sure that your firewall allows TCP connections to port 25.

List of REST API methods
------------

**Create session**
----
Returns a new session with one mailbox.

* **URL**

/api/create_session

* **Method:**

`PUT`

*  **URL Params**

None

* **Sample Request:**
```
PUT /api/create_session HTTP/1.1
```

* **Sample Response:**

```
<?xml version="1.0" encoding="UTF-8"?>
<session>
  <session_key>WmY9nvtrAr1wi4JUZRVaE3ST</session_key>
  <mailbox>
    <address>y2dxpt0@qamail.ala.se</address>
  </mailbox>
</session>

```
**List mailboxes**
----
Returns a list of mailboxes for a given session.

* **URL**

/api/list_mailboxes

* **Method:**

`GET`

*  **URL Params**

   **Required:**

   `session_key=[string]`

* **Sample Request:**
```
GET /api/list_mailboxes?session_key=G3nfwoElCc33f8ZHXEJgWflA HTTP/1.1
```

* **Sample Response:**

```
<?xml version="1.0" encoding="UTF-8"?>
<session>
  <session_key>G3nfwoElCc33f8ZHXEJgWflA</session_key>
  <mailbox>
    <address>1a7667x@qamail.ala.se</address>
  </mailbox>
  <mailbox>
    <address>e17ofvl@qamail.ala.se</address>
  </mailbox>
  <mailbox>
    <address>90mgv0o@qamail.ala.se</address>
  </mailbox>
  <mailbox>
    <address>frkjjwj@qamail.ala.se</address>
  </mailbox>
  <mailbox>
    <address>a9hn63p@qamail.ala.se</address>
  </mailbox>
</session>

```
**Create mailbox**
----
Creates a new mailbox for a given session.

* **URL**

/api/create_mailbox

* **Method:**

`PUT`

*  **URL Params**

   **Required:**

   `session_key=[string]`

* **Sample Request:**
```
PUT /api/create_mailbox?session_key=G3nfwWElCcXHf8ZHXEJgWf8A HTTP/1.1
```

* **Sample Response:**

```
<?xml version="1.0" encoding="UTF-8"?>
<mailbox>
  <address>mnjpvka@qamail.ala.se</address>
</mailbox>

```
**Show mailbox content**
----
Returns a list of letters in a given mailbox, without letter content.

* **URL**

/api/show_mailbox_content

* **Method:**

`GET`

*  **URL Params**

   **Required:**

   `session_key=[string]`
   `address=[string]`

* **Sample Request:**
```
GET /api/show_mailbox_content?session_key=Ry1Nc1wehF99t6y6DRQ8v8Uc&address=5y7yuva@qamail.ala.se HTTP/1.1
```

* **Sample Response:**

```
<?xml version="1.0" encoding="UTF-8"?>
<mailbox>
  <address>5y7yuva@qamail.ala.se</address>
  <letter>
    <id>18706</id>
    <subject>test message</subject>
    <from>v.pryakhin@gmail.com</from>
    <date>2015-04-19 14:53:59 UTC</date>
  </letter>
</mailbox>

```

**Show letter**
----
Shows letter content

* **URL**

/api/show_letter

* **Method:**

`GET`

*  **URL Params**

   **Required:**

   `session_key=[string]`
   `address=[string]`
   `letter_id=[integer]`

* **Sample Request:**
```
GET /api/show_letter?session_key=Ry1Nc1wehF99t6y6DRQ8v8Uc&address=5y7yuva@qamail.ala.se&letter_id=18706 HTTP/1.1
```

* **Sample Response:**

```
<?xml version="1.0" encoding="UTF-8"?>
<letter>
  <id>18706</id>
  <subject>test message</subject>
  <from>v.pryakhin@gmail.com</from>
  <date>2015-04-19 14:53:59 UTC</date>
  <content>Return-Path: &lt;v.pryakhin@gmail.com&gt;
Received: from mail-qk0-f170.google.com (mail-qk0-f170.google.com [209.85.220.170]) by qamail.ala.se (Postfix) with ESMTPS id 41D0D60720 for &lt;5y7yuva@qamail.ala.se&gt;; Sun, 19 Apr 2015 17:53:59 +0300
Received: by qkgx75 with SMTP id x75so168702965qkg.1 for &lt;5y7yuva@qamail.ala.se&gt;; Sun, 19 Apr 2015 07:53:59 -0700
Received: by 10.141.4.65 with HTTP; Sun, 19 Apr 2015 07:53:59 -0700
Date: Sun, 19 Apr 2015 17:53:59 +0300
From: Vitaly Pryakhin &lt;v.pryakhin@gmail.com&gt;
To: 5y7yuva@qamail.ala.se
Message-ID: &lt;CAMHkD7W01arEjYH_g-CSGpsXGD_Gu_0dB_FxQSCMRXiVLETCNw@mail.gmail.com&gt;
Subject: test message
...
...
...
end of message is skipped in the example
...
...
...
</content>
</letter>


```

**Empty mailbox**
----
Deletes all letters in a given mailbox.

* **URL**

/api/empty_mailbox

* **Method:**

`DELETE`

*  **URL Params**

   **Required:**

   `session_key=[string]`
   `address=[string]`

* **Sample Request:**
```
DELETE /api/empty_mailbox?session_key=Ry1Nc1wehF99t6y6DRQ8v8Uc&address=5y7yuva@qamail.ala.se HTTP/1.1
```

* **Sample Response:**

Response contains no body.

**JSON**
----
QA Mail API supports JSON an XML serialization. Examples above show usage of XML. To get JSON, simply add a path parameter format=json to your query.

* **Sample Request:**
```
GET /api/list_mailboxes?session_key=G3nfwoElCc33f8ZHXEJgWflA&format=json HTTP/1.1
```

* **Sample Response:**

```
{
    "session": {
        "session_key": "G3nfwoElCc33f8ZHXEJgWflA",
        "mailboxes": [
            {
                "address": "jco24sa@qamail.ala.se"
            },
            {
                "address": "v9i56eg@qamail.ala.se"
            },
            {
                "address": "c56tvej@qamail.ala.se"
            },
            {
                "address": "492xo6b@qamail.ala.se"
            },
            {
                "address": "d2syc8m@qamail.ala.se"
            },
            {
                "address": "o8cgb97@qamail.ala.se"
            },
            {
                "address": "pjegl1k@qamail.ala.se"
            },
            {
                "address": "vg6dobp@qamail.ala.se"
            },
            {
                "address": "ln1zpci@qamail.ala.se"
            },
            {
                "address": "8i2qfnp@qamail.ala.se"
            }
        ]
    }
}

```

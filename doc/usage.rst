#####
Usage
#####

This library supports creating basic GPG/Mime messages as well as basic key handling functions. It
was designed for a website that allows users to add GPG keys in order to receive GPG encrypted
emails from the website.

The common interface abstracts from different libraries, making them interchangeable. The following
example thus works with any implementation.

*********************
MIME message handling
*********************

If you are handling basic MIME messages (from pythons `email.mime
<https://docs.python.org/3.4/library/email.mime.html>`_ module), use the
:py:func:`~gpgmime.base.GpgBackendBase.sign_message` and
:py:func:`~gpgmime.base.GpgBackendBase.encrypt_message` functions::

   >>> from gpgmime import gpgme
   >>> from six.moves.email_mime_text import MIMEText
   >>> from six.moves.email_mime_multipart import MIMEMultipart

   # create backend
   >>> backend = gpgme.GpgMeBackend()

   # create message
   >>> plain = MIMEText('foobar')
   >>> html = MIMEText('html', _subtype='html')
   >>> multi = MIMEMultipart(_subparts=[plain, html])

   # get signed/encrypted/signed and encrypted message
   >>> msg = backend.sign_message(multi, signers=['your-fingerprint'])
   >>> msg = backend.encrypt_message(multi, recipients=['other-fingerprint'])
   >>> msg = backend.encrypt_message(multi, signers=['your-fingerprint'],
                                     recipients=['your-fingerprint'])

   # add various headers...
   >>> msg.add_header('From', 'user@example.com')

************************
Key management functions
************************

The interface also offers some *basic* key management. It's not very sophisticated, it's assumed
that users upload a key or give a fingerprint to be downloaded from the keyservers. Keys
can be imported, trust can be queried and set and the expiry queried::

   >>> from gpgmime.base import VALIDITY_FULL
   >>> from gpgmime.base import VALIDITY_UNKNOWN
   >>> from gpgmime import gpgme

   >>> backend = gpgme.GpgMeBackend()
   >>> fingerprint = 'E8172F2940EA9F709842290870BD9664FA3947CD'

   >>> raw_key_data = backend.fetch_key('0x%s' % fingerprint)
   >>> imported_fp = backend.import_key(raw_key_data)
   >>> imported_fp == fingerprint
   True

   >>> backend.get_trust(fingerprint) == VALIDITY_UNKNOWN
   True
   >>> backend.set_trust(fingerprint, VALIDITY_FULL)
   >>> backend.get_trust(fingerprint) == VALIDITY_FULL
   True

   >>> backend.expires(fingerprint)
   datetime.datetime(2017, 10, 8, 21, 14, 53)

******************
Django integration
******************

Since everyone (including me) seems to use `Django <https://www.djangoproject.com/>`_ nowadays,
this library also integrates with Django.

Use :py:class:`~gpgmime.django.GPGEmailMessage` instead of
`EmailMessage <https://docs.djangoproject.com/en/dev/topics/email/#emailmessage-objects>`_
objects::

   >>> from gpgmime import gpgme
   >>> from gpgmime.django import GPGEmailMessage

   >>> backend = gpgme.GpgMeBackend()
   >>> fingerprint = 'E8172F2940EA9F709842290870BD9664FA3947CD'

   >>> msg = GPGEmailMessage(subject='subject', ...,
                             gpg_recipients=[fingerprint], gpg_signers=[fingerprint])
   >>> msg.send()
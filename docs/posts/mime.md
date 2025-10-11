---
date:
  created: 2024-07-22
pin: true
links:
  - Homepage: index.md
categories:
  - Software
tags:
  - internet standards
  - internet
authors:
  - jacob
slug: mime
---

# MIME

Strap in, this might just be the most thrilling blog post ever written, because today I am writing about internet standards.

Multipurpose Internet Mail Extensions (MIME) is an internet standard specified in a series of Request for Comments (RFC) written by the Internet Engineering Task Force. It was proposed by Nathaniel Borenstein in 1991, to allow people to send and receive emails that contained images, videos, documents and non-ASCII characters – šüçħ åş ťħęßē fâňčÿ ċħàřáćțęřş. It also allows people to transport different data representations in a single message – such as plain text and HTML – and does not impose restrictions on the length of the message.

Although, MIME was designed mainly for the Simple Mail Transfer Protocol (SMTP), it is important in other communication protocols. For example, in HTTP, a MIME header is inserted at the beginning of any web transmission. The purpose of MIME is to ensure interoperability across different email clients and systems (such as Gmail, Yahoo, Outlook, etc.). In fact, if you view the raw source of an email, you’re guaranteed see many of MIME header fields described in this post.

<!-- more -->

## Basic Text Email

Let’s start with an example of a MIME message for a basic email.

```
MIME-Version: 1.0
From: Alice <alice@example.com>
To: Bob <bob@example.com>
Subject: Free a quick call?

Content-Type: multipart/alternative; boundary=“frontier”

--frontier
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

Hi Bob, 

Are you be available for a quick call on Wednesday?

Thanks,
Alice

--frontier
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

<html>
   <body>
      <p>
         <br>Hi Bob
         <br>Are you be available for a quick call on Wednesday?
      </p>
      <p>Thanks,</p>
      <p>Alice<br>
   </body>
</html>

--frontier--
```

The message contains a plain text and an HTML part arranged as part of a tree structure with, the C`ontent-Type: multipart/alternative` part being the tree’s head and the `Content-Type: text/*` as the leaf nodes.

Let’s break down each header to understand what is going on:

### MIME-Version: 1.0

This has the dual purpose of telling the email client that the message leverages MIME to format the email as well as the version number. Although this versioning was introduced to permit changes to the MIME protocol, the creators didn’t specify how to handle future MIME versions, and thus this header almost always reads 1.0.

### From, To & Subject

Fairly straightforward, these headers define who the message sender and recipients are as well as the email subject.

### Content-Type

The Content-Type header indicates the media type of the message content. This could be text, image, audio, video and so on. In our message above we have three Content-Type headers: **multipart/alternative**, **text/plain** and **text/html**. The Content-Type header field consists of a type and a subtype separated by a slash. In the email above the **multipart/alternative** Content-Type indicates to the client that each subsection is an “alternative” version of the same or similar content. We can verify this as the text and HTML subsections (separated by a boundary) contain the exact same text. As such our email provides backwards compatibility while the HTML part allows us to introduce nicer formatting, such as hyperlinks or images.

The `boundary=“frontier”` specifies that each subsection of the multipart/alternative content type is separated by the string “--frontier”. Normally, you wouldn’t use a word as a boundary and instead use a random string to ensures that the boundary does not accidentally appear in any parts it shouldn’t.

### Content-Transfer-Encoding

Lastly, the Content-Transfer-Encoding header indicates the binary-to-text encoding of the part. In order to transfer data via email (for example an image), we first need to extract it’s binary representation then transform it to text. There are several ways to do this, but the two used in the MIME protocol are quoted-printable and base64. These tell the email client what binary-to-text encoding scheme was used, and that initial decoding is needed before the message can be read.

## Email With Attachment

Now let’s look at a slightly more complicated example of a MIME message that contains a PDF attachment.

```
MIME-Version: 1.0
From: Alice <alice@example.com>
To: Bob <bob@example.com>
Subject: See attached.
Content-Type: multipart/mixed; boundary="main-section"

--main-section
Content-Type: multipart/alternative; boundary="sub-section"

--sub-section
Content-Type: text/plain; charset="UTF-8"

Attached is a PDF.

Best wishes,
Alice

--sub-section
Content-Type: text/html; charset="UTF-8"

<html>
   <div>
      <div>Attached is a PDF.
      </div>
      <div><br></div>
      <div>Best wishes,</div>
      <div>Alice</div>
   </div>
</html>

--sub-section--
--main-section
Content-Type: application/pdf; name=“document.pdf"
Content-Disposition: attachment; filename="document.pdf"
Content-Transfer-Encoding: base64

JVBERi0xLjcNCiW1tbW1DQoxIDAgb2JqDQo…
--main-section—
```

There are a couple of new headers here. First, the example introduces the **multipart/mixed** type and subtype of the Content-Type header. This is used for sending parts with different Content-Type header fields. As you may have spotted, we have some nesting going on here. This is the tree structure I alluded to earlier in action.

The “sub-section” boundary divides the parts of the multipart/alternative content type, i.e. the text and the HTML content. While the “main-section” boundary divides the parts of the multipart/mixed content type, i.e. the email body and the PDF.

### Content-Disposition

The Content-Disposition header underneath the application content type header specifies how to present the attachment. There are two valid dispositions: inline, which denotes that the media file should be displayed as soon as the message is displayed, and attachment, which specifies not to display the media file, and instead wait for some action by the user (usually click on the file).

In addition, the Content-Disposition field also provides parameters for specifying the name of the file, the creation date 
and the modification date.

# Additional Headers

There are several more headers I have not yet mentioned that typically get added to MIME messages, including:

| Header        | Description                                                                                           |
|---------------|-------------------------------------------------------------------------------------------------------|
| Date          | The date the email was sent.                                                                          |
| Message-ID    | A unique identifier that helps email clients organise emails into threads.                            |
| Content-ID    | A unique identifier of any media files that were added to the message, so they can be easily referenced by the email client. |
| Reply-To      | The email address to send a reply to (might be different from the original sender).                   |
| In-Reply-To   | The ID of the message that the current email is responding to.                                        |
| Cc            | A list of recipients that receive a carbon copy of the message.                                       |
 
There are a few more that you might encounter if you looked at the raw source of an email – such as the **DKIM-Signature** – but in the interest of time I’ll leave it at that (there are six RFCs which specify the MIME protocol, some of which are over 100 pages long).

Finally, you may also encounter headers that start with X– (x dash). These are unofficial “experimental" header fields that are typically added by the email client for internal use.

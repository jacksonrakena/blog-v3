---
layout: post
title: "custom email on a budget: MXroute vs Migadu"
tags: email economy
---

*This post is unfinished, but I thought I'd release it.*  
  
  

Today you're gonna be getting a rundown of the 'Rakena Email System' - and how stingy I truly am when it comes to paying for email services. (And maybe some pretty sketchy economic analysis.)

### Why do you care so much about email addresses?

As some of you might know by now, I run a private mail setup for my family - using
the `rakena.co.nz` domain, which I own. (There's some bootleggers too, like `quicktable.net`,
`greemdev.net`, and `abyssal.gg`)

I was always attracted to the idea of professional, private email - being able to send email as `jackson@rakena.co.nz`, instead of, say, `jacksonrakena@gmail.com`. It suggests that the sender is important/professional enough to have a custom domain. And my dad likes it too for his business communications. A double ego win!

Normally, most companies looking for email would have it provided by their internal IT departments, usually in the form of something like [Google Workspace](https://workspace.google.com/) (formerly G Suite), or, in the case of my high school and my (current) workplace, [Microsoft 365 (Exchange)](https://www.microsoft.com/en-nz/microsoft-365). Unfortunately, these are both some pricey options, with them both clocking in at around $4-8 New Zealand dollars per month.

So instead, I'm going to make a comparison between a few small providers, namely [Migadu](https://migadu.com) (which I use at the moment), [MXroute](https://mxroute.com) (a rather worthy competitor), and the aforementioned options.

## The criteria

The criteria I'm looking for in my dream mail provider are summed up in this list, in order of how much I care:

- Price
  - Preferred: Bulk storage-based pricing (i.e. $X/month for Y GB, instead of $X per user, per month)
- Features
  - Webmail client (web access, ease-of-use, style)
  - Federation with SAML/OIDC single-sign-on (for use with my in-house IAM provider, [Rakena IAM](https://iam.rakena.co.nz))
  - Out-of-office (holiday) autoreplies
  - Quality and speed of spam filtering
  - Setup: How long it takes to add an email account to my family members' devices (iPhones especially, Windows and Mac laptops)
- Security
  - DMARC, DKIM, SPF, the usual suspects.
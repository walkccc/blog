+++
date = 2026-07-30T00:00:00-04:00
title = "One Inbox, Many Masks: How I Run Multiple Products With One Google Workspace"
tags = ["Indie", "Marketing", "Ops"]
categories = ["Essay"]
+++

I currently run three products — [zestimer.com](https://zestimer.com), [furioke.com](https://furioke.com), and [playtefuda.com](https://playtefuda.com) — plus my studio domain, [magicparklabs.com](https://magicparklabs.com). At first, I thought every product should have its own Google Workspace account. Then I checked the price. Google Workspace charges by user, not by domain, and for a one-person indie studio, creating another user just because I want `contact@newproduct.com` feels unnecessary. So I ended up putting all my domains into one Workspace account.

The basic idea is actually very simple: **a product needs an email address, but it doesn't necessarily need another user account.** For example, let's say I just launched `newproduct.com`. I want to have `contact@newproduct.com`, but I don't want another Gmail inbox, another Workspace user, or another bill. What I really want is for both `contact@magicparklabs.com` and `contact@newproduct.com` to go into the same Gmail inbox, while still being able to send and receive emails as the new product.

This is how I set it up. There are basically two places involved: [Cloudflare DNS](https://dash.cloudflare.com) and [Google Admin Console](https://admin.google.com). The most important thing to remember is the order: **update DNS first, then go back to Google and confirm.** A lot of Google's verification screens are basically saying, "We are waiting for you to publish this DNS record." Once I understood this, the whole process became much less confusing.

## Add the domain to Google Workspace

I use **Cloudflare** for domain registration, so when I buy a new domain, the DNS is already there.

1. Open [Google Admin Console](https://admin.google.com)
1. Go to **Account → Domains → Manage domains → Add a domain**.
1. Enter `newproduct.com`, choose **Secondary domain**, and click **Add domain & start verification**.

This part is important: don't choose **Domain alias**. A domain alias basically creates another address for every existing user. So if you already have `contact@magicparklabs.com`, a domain alias would make `contact@newproduct.com` part of the same mailbox identity. That's not really what you want. You want each product to have its own namespace while still having everything delivered to my one inbox, so let's use **Secondary domain**.

## Verify the domain

Google will take you to **Let's set up your domain**.

1. Click **Get started**.
1. Under domain verification, click **Sign in to Cloudflare**. Since the domain is already managed by Cloudflare, Google can add the verification record for you.
1. Cloudflare will open a page called **Authorize DNS records from Google**. You'll see the DNS record Google wants to add. It should look roughly like this:

   | Type | Name | Content | TTL | Proxy status |
   | --- | --- | --- | --- | --- |
   | `TXT` | `newproduct.com` | `"google-site-verification=…"` | 1 hr | DNS only |

1. Click **Authorize**. This gives Google permission to add the verification record to your Cloudflare DNS.
1. You'll be sent back to Google. It will show **Getting your domain ready** while checking the DNS record. Just wait. A few minutes is normal.
1. Once everything is set up, Google should show **Your domain is verified!**

At this point, Google knows that you own the domain. Email isn't working yet, though. That's a separate step.

## Activate Gmail

From **Your domain is verified!**, click **Activate Gmail**.

1. Click **Start using Gmail with your domain → Continue**.
1. Google will ask which users should receive mail for the new domain. If you're an indie hacker, it's just you.
1. Click **Proceed with activation**.
1. You'll then see **Add Gmail activation code**. This is where the MX record comes in. In Cloudflare, the record should look like this:

   | Type | Name | Mail server       | Priority | TTL   |
   | ---- | ---- | ----------------- | -------- | ----- |
   | `MX` | `@`  | `smtp.google.com` | `1`      | 1 min |

   One important thing: **delete the other MX records for `@`.** You might have old records from your registrar, email forwarding, domain parking, or something else. Mail delivery follows the MX records, so leaving random old ones around can create some very confusing behavior.

1. Open Cloudflare and add or update the MX record.
1. Delete any old MX records for `@`.
1. Go back to Google.
1. Tick **Come back here and confirm once you have updated the code on your domain host**.
1. Click **Confirm**.

One small thing that's easy to miss: Google's **Go to Cloudflare** button doesn't actually publish the record for you. It just opens Cloudflare, so you still need to make the DNS change yourself. Eventually you'll see **Gmail is now ready**. Google says it can take up to 24 hours for everything to fully propagate, although in my experience it usually starts working much sooner.

## Set up DKIM

This is the one I wouldn't skip.

DKIM signs outgoing emails and helps receiving mail servers verify that emails really came from your domain. For a brand-new product domain, I don't want the first few emails I send to immediately look suspicious or end up in spam.

In Google Admin Console:

1. Go to **Optional setup steps → Authenticate outgoing emails**
1. Click **Add verification key** - Keep **Key bit length: 2048**
1. Click **Generate key**
1. Copy the **Name** and **Content** Google gives you
1. Don't click the confirmation checkbox yet

Then go to Cloudflare:

1. Open **DNS → Records → Add record**
1. Add a TXT record like this:

   | Type  | Name                | Content                   | TTL   |
   | ----- | ------------------- | ------------------------- | ----- |
   | `TXT` | `google._domainkey` | `v=DKIM1;k=rsa;p=MIIBIj…` | 1 min |

1. Save the record
1. Go back to Google
1. Click the confirmation checkbox

The order matters: Google generates the key, you publish it to DNS, then Google checks whether it can find it. If you confirm before adding the DNS record, Google will simply tell you it can't find the key. Which, to be fair, is correct.

## Add the product email address

Now the domain can receive email, but we still need to tell Google which user should receive it.

In Google Admin Console:

1. Go to **Directory → Users**
1. Click your own name, not the checkbox
1. Go to **User information → Alternate email addresses (email alias)**
1. Click **ADD AN ALTERNATE EMAIL**
1. Enter `contact`
1. Choose `newproduct.com` from the domain dropdown and click **Save**

That's it. You now have both `contact@magicparklabs.com` and `contact@newproduct.com` arriving in the same Gmail inbox.

## Send emails as the product

Receiving email is only half of it.

If someone emails `contact@newproduct.com`, I also want to reply from `contact@newproduct.com` instead of accidentally replying from my personal or studio address.

In Gmail:

1. Go to **Settings → Accounts and Import → Send mail as**
1. Add the new address
1. Keep **Treat as an alias** checked
1. Under **When replying to a message**, choose **Reply from the same address the message was sent to**

Now the whole thing feels like a separate product inbox, even though technically everything is still sitting inside the same Gmail account. This is probably my favorite part of the setup. From the user's perspective, `contact@newproduct.com` is a real email address. From my perspective, it's just another mask on the same inbox.

## Optional: SPF

Cloudflare will probably start reminding you about SPF.

You can find the recommendation under **DNS → Records → Recommendations**. Look for **Prevent unauthorized email senders**.

Cloudflare's **Email Record Creator** makes this pretty painless:

1. Leave **IP addresses** empty
1. Put `_spf.google.com` under **Domains**
1. Keep the policy as **Soft fail: ~all**
1. Cloudflare should generate something like:

| Type  | Name             | Content                                 | TTL  |
| ----- | ---------------- | --------------------------------------- | ---- |
| `TXT` | `newproduct.com` | `"v=spf1 include:_spf.google.com ~all"` | Auto |

One thing to watch out for: **a domain should only have one SPF record.** If you already have an SPF record, don't create another one. Edit the existing record and add Google's `include` instead.

## Optional: DMARC

Cloudflare also has a recommendation for DMARC: **Block fake emails sent from @newproduct.com addresses**.

Go into the DMARC section and enable **DMARC Management**. Cloudflare can create the record and collect the reports for you.

I don't think you need to spend hours learning email authentication before launching your product. But SPF, DKIM, and DMARC are worth setting up once you start sending real product emails.

## Optional: Automatically label each product

One more thing I like doing is giving each domain its own Gmail label.

In Gmail:

1. Go to **Search → filter icon → To**
1. Enter `newproduct.com`
1. Click **Create filter → Apply the label → New label**
1. I usually name it something like `[NewProduct]`

Do this when you add the domain, not six months later when the inbox has become a complete mess.

## That's basically the whole setup

One Google Workspace user, multiple domains, multiple `contact@` addresses, and one Gmail inbox. For an indie developer running several small products, I think this is a much better setup than creating a new Workspace account every time you launch something. The domains are separate, the products are separate, and the email addresses are separate, but I only have one inbox to worry about. **One inbox, many masks.**

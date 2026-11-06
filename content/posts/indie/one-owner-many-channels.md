+++
date = 2026-08-13T00:00:00-04:00
title = "One Owner, Many Channels: Short-Video Accounts per Product"
tags = ["Indie", "Marketing", "Ops"]
categories = ["Essay"]
+++

[One Inbox, Many Masks](/posts/indie/one-inbox-many-masks/) gave every product its own `contact@` address inside a single Google Workspace seat. Opening the first YouTube Shorts channel for [Zestimer](https://zestimer.com) asked the follow-on question: sign up as `contact@zestimer.com`, or as `contact@magicparklabs.com`?

Both — and which does which turns out to depend on the platform.

> The address that **owns** the account should belong to the studio. The address the **audience sees** belongs to the product.

The public address is a profile field: editable any day, on every platform, without touching the account. The login is not. It's the recovery path, the 2FA anchor, and the thing a platform asks about when an account is locked or disputed. Products get renamed, sunset, or sold; the studio outlives them, which is why ownership wants to sit there.

Only one of the three platforms actually lets you arrange that.

| Platform  | Signs in / owns             | Shown on the profile   |
| --------- | --------------------------- | ---------------------- |
| YouTube   | `contact@magicparklabs.com` | `contact@zestimer.com` |
| TikTok    | `contact@zestimer.com`      | `contact@zestimer.com` |
| Instagram | `contact@zestimer.com`      | `contact@zestimer.com` |

## YouTube: one Google account, many channels

A Brand Account is exactly this separation, built in. One Google account can own several channels, each with its own name, handle, and branding, and none of them expose the underlying account's identity.

Signed in as `contact@magicparklabs.com`: **youtube.com** → avatar → **Settings** → **Add or manage your channel(s)** → **Create a channel**. Give it the product's name and handle. Naming it anything other than the Google account's own name is what makes it a Brand Account.

**Do not rename the studio channel into the product channel.** If a `MAGIC PARK LABS @MagicParkLabs` channel already exists, leave it alone and create a second one. The studio channel is the parent, not the first draft of the product channel.

The public address then goes in **YouTube Studio** → **Customization** → **Basic info** → **Email for business inquiries**. That field is what viewers reach through the **View email address** button on the channel's About panel, behind a captcha. `contact@zestimer.com`.

Extra managers, later, are added under the Brand Account's permissions — by Google account, without sharing a password.

## TikTok and Instagram: the product address does both jobs

Neither platform has a Brand Account equivalent, and both bind an email address to exactly one account. The studio address is spent the moment the studio's own account uses it, so it cannot open the product's — the separation YouTube hands you for free isn't on the menu here. `contact@zestimer.com` signs in, and `contact@zestimer.com` is what the profile prints.

That publishes the login, and it parks account recovery on a domain tied to a single product. Both are real, and both are smaller than they look. What stops a takeover is two-factor auth, not a secret username: a known address buys an attacker phishing attempts and reset-mail noise, none of which survives an authenticator app. And if Zestimer is ever sold, the domain, the mailbox, and the channels were always moving as one bundle — sharing an address makes that handover cleaner rather than messier.

So it's fine, on two conditions — and they're conditions worth meeting either way:

- **Two-factor with an authenticator app, not SMS.** This is the actual lock, and it's the one that makes a public login a non-event.
- **Keep `zestimer.com` on auto-renew.** The single scenario that really costs you the account is a live channel behind an expired domain: whoever registers it next receives your password resets. Cloudflare's registrar auto-renews by default — confirm it rather than assume it.

If you'd rather not print the login, one Workspace alias — `zestimer@magicparklabs.com`, added like any other ([step 6](/posts/indie/one-inbox-many-masks/#6-admin-console-add-the-address-to-your-one-user)) — buys the separation back, and that's the shape most companies use: an internal ops address signs in, the public one only ever appears on the profile. For a one-person studio the authenticator app is already doing that work.

**TikTok.** The public field exists only on a Business Account — **Settings and privacy** → **Account** → **Switch to Business Account**, then **Edit profile** → **Email**. TikTok mails a confirmation link to whatever you type there. If anyone else ever touches the account, add it as an asset in **Business Center** rather than handing over the login.

Signup asks for a date of birth: use your real one. It's the operator's, not the brand's, it never appears on the profile, and it gates LIVE, ads, and monetization at 18+. Typing the product's launch date reads as a newborn account. It's also near-immutable afterwards — correcting it means a support request with government ID, which is the same document every recovery and payout check compares against.

**Instagram.** Switch to a Professional account, then **Edit profile** → **Contact options** → business email → `contact@zestimer.com`. It stays hidden until **Edit profile** → **Profile display** → **Display contact info**. Leave phone and address empty and the resulting button reads **Email** instead of the generic **Contact**.

## The shape it leaves

```
Magic Park Labs
│
├── contact@magicparklabs.com
│   └── YouTube ── @MagicParkLabs, @Zestimer, @Furioke   (Brand Accounts)
│
├── Zestimer ── zestimer.com
│   └── contact@zestimer.com ── TikTok @zestimer, Instagram @zestimer
│                            └─ public contact on all three channels
│
└── Furioke ── furioke.com
    └── contact@furioke.com ── TikTok @furioke, Instagram @furioke
                            └─ public contact on all three channels
```

Every address in that tree lands in one inbox. To the audience, each product is its own company with its own contact; to me, it's one login screen and one authenticator app.

None of it is permanent, either. Every platform here lets you change the account's email later — it's the one field they're relaxed about, unlike the birthday — so a product that outgrows the arrangement can be moved onto its own ownership address the day that matters. What you can't redo cheaply is the follower count, which is the argument for opening the product's channel as the product's, rather than renaming the studio's later.

Same trade as the inbox, one layer up: the masks are cheap, the account underneath is the asset.

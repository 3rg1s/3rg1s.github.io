---
title: My First Bug Bounty Experience
date: 2025-04-18 16:41:01
categories: ["other", "bug bounty"]
banner_img: /img/hackerone.png
---

Stumbling upon my first security vulnerability was a wild ride—part luck, part persistence. I wasn’t expecting much, but it ended with a triple-digit bounty and a huge adrenaline rush. Here’s how it all played out.

Due to confidentiality agreements, I cannot disclose the specific program where I identified the vulnerability.

### First Attempt: A Duplicate Discovery

My curiosity led me to examine the application's behavior on my mobile device, particularly its network requests. To facilitate this, I set up Burp Suite, installed its certificate on my phone, and configured the proxy settings to route traffic through my computer for interception.

One feature of the application allowed users to send reminders to their friends, seemingly restricted to friends only. By intercepting the request responsible for this action, I modified the user ID parameter to target another user. User IDs were accessible in various parts of the application, though I cannot provide further details to maintain confidentiality.

Testing revealed that the endpoint lacked rate limiting and failed to validate whether the target user was actually a friend. This oversight allowed me to send reminders from any user to any other user, bypassing the intended restrictions entirely.

However, when I reported this issue, the program informed me that the bug had already been identified, marking it as a duplicate.

### Second Attempt: Another Duplicate

Undeterred, I continued my search and discovered another endpoint with a similar vulnerability. By modifying the user ID parameter, I was able to trigger a flood of reactions on a user's feed. This behavior mirrored the previous issue, as the endpoint again failed to validate the user ID properly.

Unfortunately, this bug was also marked as a duplicate upon submission.

### Third Attempt: A Valid Discovery

After two rejections, I was determined to uncover a valid bug. This time, I shifted my focus from intercepting requests on my mobile device to analyzing browser requests directly on my computer. During this process, I identified an endpoint that revealed various pieces of information when provided with a player's ID.

#### Roles and Early Users

The player IDs were numeric and sequential, allowing me to access data associated with early-stage users of the platform, such as IDs 1, 2, 3, and so on. The endpoint returned something particularly intriguing: the roles assigned to each user. As expected, the earliest users—likely the platform's developers—had the most extensive roles assigned to them.

#### Exploiting Role Manipulation

While my roles differed from those of, say, the user with ID 1, I couldn't directly write to the database to assign myself those roles. Instead, I devised a clever workaround: intercepting all responses that included my roles and modifying them to reflect the roles of the user with ID 1. This tricked the frontend into believing these were the roles provided by the backend.

#### Accessing Internal Tools

By doing this, my requests to the website were automatically redirected to a subdomain of the main website. On this subdomain, I gained access to various internal tools primarily designed to assist frontend developers. Additionally, I discovered other subdomains that exposed sensitive data not intended for external access.

### Reporting and Bounty

I put together a detailed report outlining the bug and highlighted how things could’ve gone south if someone with bad intentions had found it first. The team got back to me quickly, marked it as a medium severity issue, and sent over a $500 bounty. It ended up being classified as an improper access control vulnerability.
---
layout: post
title: "My First Week: 3 Business Logic Bugs in Major E-Commerce"
date: 2026-02-03
tags: [bug-bounty, business-logic, web, burp-suite]
canonical_url: https://medium.com/@youalialassaf/my-first-week-3-business-logic-bugs-in-major-e-commerce-8b303e02771f
excerpt: "My first week of bug bounty hunting: three logic flaws on a large e-commerce site, why they were closed as Informative, and what impact really means."
---

*Originally published on [Medium](https://medium.com/@youalialassaf/my-first-week-3-business-logic-bugs-in-major-e-commerce-8b303e02771f). The target is anonymized as "Target-X"; no vulnerable data is shown.*

## Introduction: hunting for broken logic

Last week I started my journey in bug bounty hunting. Instead of chasing complex code vulnerabilities such as RCE or SQLi, I decided to focus on something that needs human thinking: business logic errors.

I picked a target, a large European e-commerce site I'll call "Target-X," and fired up Burp Suite. I wanted to see if I could break the rules the developers set. Within a week I found three places where I bypassed the site's restrictions. Here is what I found.

## Bug #1: Bypassing the minimum order value

**The rule:** you cannot check out if your cart total is under $50. The checkout button is greyed out in the browser.

**The approach:** I asked whether the rule was enforced by the server or only by the frontend. I added a $5 item, saw the disabled button, then typed the checkout URL directly into the address bar: `site.com/checkout/payment`.

**The result:** the server did not re-check the cart total. The payment page opened for a $5 order, ignoring the $50 rule.

## Bug #2: The shipping fee trick

**The rule:** free shipping only for large orders; small orders pay a fee.

**The approach:** using forced browsing, I jumped straight to the payment page with a cheap item. Because I skipped the normal steps (address, then shipping method, then payment), the backend never applied the shipping fee at that stage.

**The result:** I reached the final payment screen with a cheap item and $0 shipping, which should have been impossible for a small order.

## Bug #3: Exploiting a promo code

**The rule:** a coupon gave 20% off only if you bought Item A **and** Item B together.

**The approach:** I added both items to activate the discount, went to checkout, intercepted the request in Burp, and removed Item B from the request just before the server processed it.

**The result:** the server processed the order for Item A only but kept the discount, so I got the discount on a single item without meeting the condition.

## The outcome: a hard lesson

I wrote reports for all three and submitted them. I was confident because I had clearly broken the site's logic. The security team closed all three as:

> **Status: Informative** (closed without bounty).

**Why?** The response was fair but tough:

> "You successfully bypassed the UI, but you did not prove the final financial impact. The backend payment system might reject the order after you click Pay, or the warehouse system might flag it later."

## What I learned

- **Logic is weak:** developers often trust the frontend too much.
- **Impact is everything:** next time I won't stop at the payment page. I need to prove, safely and within scope, that the company actually loses money before a report is accepted.

It was a solid start to my bug bounty journey.

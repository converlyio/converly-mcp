---
name: setup-conversion-tracking
description: Set up Converly conversion tracking end to end, from the user's goal to a published flow, an installed snippet and a verified conversion. Use when the user wants to track form submissions or bookings as conversions, send leads to an ad platform like Google Ads, Meta or Google Analytics, or set up Converly for the first time.
---

# Set up conversion tracking with Converly

Aim for a published flow as quickly as possible, then get the snippet onto the website. Tracking only works when FOUR things are all true. A flow is published, the ad platform is ready (connected where it needs connecting, and its own tag on the website where that is required), the site has its website address set, and the Converly snippet is installed on the website. Never end this workflow without all four done or clearly handed to the user.

## Steps

1. **Understand the goal.** Ask which form or booking tool the site uses (Typeform, Webflow forms, Gravity Forms, Calendly and so on), which ad platform conversions should go to, and which page the form is on or which forms should count, unless the user already said. Most sites have more than one form. If the tool isn't in `list_trigger_types`, say plainly that it isn't supported through this connection. Never improvise a workaround.

   While you ask, quietly call `get_subscription` and `list_sites`. If the account has no active trial or subscription, every change will be refused, so tell the user now to sort out their plan in Converly at app.converly.io, rather than halfway through. If there is more than one site, ask which website they mean and pass its `site_id` to every tool that takes one.

2. **Create the draft flow.** Call `create_flow` straight away with just a name that describes the goal (for example "Contact form to Google Ads"). A draft fires nothing until it is published, and it is saved in the user's Converly dashboard if they stop part way.

3. **Add the form tool.** If the tool's entry in `list_trigger_types` has `requires_connection` true (Typeform, Calendly and similar), connect it first: call `connect_trigger_source`, share the link, explain the account owner must open it in their browser and sign in, and ask them to tell you when they've finished. Confirm with `check_handoff_status`, then pick the specific form or event type from `list_trigger_filter_options`, never invented values. Tools Converly detects on the page itself (Gravity Forms, Webflow forms and the rest) need no connection; narrow those to the page the form is on, using its exact path. Add the trigger to the flow with `update_flow`.

4. **Connect the ad platform.** Find the platform in `list_destination_types` and read its setup notes. They say in plain words whether it must be connected first, and whether the platform's own tag (the Meta Pixel, the Google tag, the Snap Pixel and so on) has to be on the website. If it must be connected and `list_destinations` shows it isn't, call `connect_destination` and share the link. Whoever has access to the ad account must open it, which may not be the person you are talking to, so say they can forward it and that it expires. When they say they've finished, confirm with `check_handoff_status`, then tell the user which ad account or pixel was connected so they can catch a wrong one. If the user wants more than one ad platform, do this for each. One flow can send to several.

   Then cover the platform's own tag. Where the notes say it is required, nothing is delivered without it, so ask whether it is already on the site and treat it as a job still to do if it isn't. Where it is recommended, say tracking works without it but results are better with it.

5. **Choose the conversion and add the action.** Every platform needs to know which conversion to record. Where the platform's notes say a conversions list is available, call `list_conversions`. On most platforms it is a short menu of standard event names such as Lead, so suggest the one that fits the goal and confirm it. On platforms where conversions live inside the ad account it lists the real ones, so agree one with the user and never pick one the list marks as not usable. If nothing fits and the list says a new one can be created, offer `create_conversion`, but only after the user agrees, because it changes their real ad account. Where no list is available, the event name is typed in, so ask the user for the exact name set up on that platform. Then call `list_action_types` for the exact settings the action needs and add it to the flow with `update_flow`.

6. **Check the flow.** Run `validate_flow`. It returns problems and warnings, and both matter.
   - **Problems** block publishing (a missing form tool or action, a missing setting, an ad platform that isn't connected, a conversion that can't be used). Fix each one.
   - **Warnings** don't block publishing, but each one means conversions won't arrive. If the website address isn't set, ask for it and set it with `update_site` straight away. Converly rejects conversions from websites it doesn't recognise. If the form tool needs a step on its own side, pass the instructions on to the user. Keep anything that needs the user for the wrap-up list. A warning that the snippet hasn't been seen is expected at this point and is handled after publishing.

7. **Publish.** Call `publish_flow`. Nothing fires until the flow is published.

8. **Get the snippet installed.** Call `get_install_status`. Unless detection is "confirmed", call `get_install_snippet` for the script tag, then choose the easiest way to get it onto the site. This step needs someone to act, so never skip it.
   - **If you can add it yourself, offer to.** If this conversation has a connector for the user's website or Google Tag Manager (a Webflow or WordPress connector, for example), or you are working in the website's code, offer to add the snippet. Ask first, use only access you already have, and never ask for logins or passwords. Say clearly if the site or the Tag Manager container still needs publishing or deploying before the change is live, and leave that to the user unless they ask you to do it.
   - **Otherwise give the user the script tag and the steps for their platform.** Work out what the website is built on. The form tool often tells you, and ask if it doesn't. The snippet goes in the site's head section, through the website platform's custom code setting, or in Google Tag Manager as a Custom HTML tag that fires on all pages. Share the matching guide: `https://help.converly.io/installing-converly/install-on-<platform>` exists for `google-tag-manager`, `wordpress`, `webflow`, `framer`, `wix`, `squarespace` and `duda`, and `https://help.converly.io/installing-converly/install-converly` covers everything else. The guides start by copying the code from Converly, which the user can skip because you have just given it to them.
   - **If someone else runs the website,** offer to write a short message the user can forward, with the script tag, where it goes and the guide link.

9. **Verify with a real submission.** Once the snippet is in place, ask the user to submit the form on their website the way a visitor would. Then call `list_events`. Allow a minute, and a little longer for tools that were connected with a sign-in, such as Typeform or Calendly.
   - **The conversion is there.** Call `get_event_detail` and tell the user the result at each ad platform. This is the proof that it works end to end.
   - **The conversion isn't there.** Call `get_install_status` to find out why.
     - "confirmed" with source "heartbeat" means Converly is running on the site, so the snippet is fine and the form wasn't picked up. Check the flow uses the right form tool and the right page or form, and that the submission really went through.
     - `origin_authorized` false means the website address on the site is wrong. Fix it with `update_site` and ask for another submission.
     - "never_seen" means the snippet isn't live yet. Website builders usually need a republish, and caching plugins may still be serving the old page. See the section below.

10. **Wrap up.** End with a short list of the four requirements, each marked done or still to do (including the ad platform's own tag where it is required or recommended), and who needs to do anything that is left (the user, their web developer, or whoever owns the ad account).

## When the signals don't show up

- **Tag managers and cookie banners.** If the snippet goes through Google Tag Manager or waits for cookie consent, the code only runs after that tag fires or the visitor accepts. Test after accepting cookies. Visitors with privacy signals, or who decline cookies, never show up as "seen".
- **Caching and speed plugins (WP Rocket and similar).** Their JavaScript minify and combine settings can break the snippet. Ask the user to exclude the Converly script from those settings, then clear the site cache and test again.
- **Converly Webflow app.** Sites set up through the Webflow app don't report as "seen". A real form submission in `list_events` is the proof instead.

## Cautions

- `send_test_event` only checks that Converly can deliver to the ad platform. It does not show up in `list_events` and does not prove the website is set up. On a server side ad platform without a sandbox test code it records a REAL conversion, so use it only when the user agrees.
- If a change is refused for billing, the account has no active trial or subscription. Tell the user to sort out their plan in Converly, then continue.
- Never guess ids. Sites, flows, destinations and events all come from the list tools, with prefixed ids.

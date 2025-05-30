---
layout: default
title: "Android 15 enhanced App Links is an opportunity to give a user an excellent experience"
date: 2025-01-29
---

### Android 15 enhanced App Links is an opportunity to give a user an excellent experience

![Featured image for "Android 15 enhanced App Links..."](/assets/1*b6AzImEA43XyjdHhOQ1YNg.png)

In the latest Android 15 update, developers are greeted with enhanced control over App Links. It’s introducing a level of flexibility and precision that was previously unattainable. Let’s talk quickly about that.
For the example and summary please jump at the end of the article.

Software Engineers now have the power to *exclude* or *allow* app links.
It’s something new, delivering fine grained experience - as you can imagine disabling/enabling an entire group of app links or *a very specific* *scenario*
is now available to us.

On top of the above, beyond the traditional `<data>` attributes,
Android 15 extends its functionality to allow filtering based on:

*   a query parameter and the corresponding value,
*   fragment,
*   and path prefix;

The granular approach Google introduced to app link management ensures that a user is taken to the appropriate resources in your application.
Mixing that with accurate data being passed as query params, in example,
a user can see improvements from enhanced filtering and smoother experience, to less bandwidth heavy and more narrowed
API calls! As long as your code can handle it. ;)

Embracing these updates, developers can craft more refined and targeted app experiences, while users enjoy more customised interaction with their apps.

---

No more talking!

![No more talking GIF](/assets/1*i5SnphHIujLd3oz9CIrDfA.gif)

---

Lets dive into some details and examples.

![Review the new stuff. The work is mine.](/assets/1*_NzlS8Vr5mhbBRsD-KnLMw.jpeg)
*Review the new stuff. The work is mine.*

Having in mind the fact, that nothing is perfect, including the schema and the designs on URL/URI in every company, the new App Links capabilities could save us from impossible scenarios.

I will omit the path prefix examples, since it’s quite obvious, and on top of that, we all had *pathPattern* already defined in the `<data>` attributes.
Please see right hand side of the above image.

I would argue that a mix of allow/block lists and the query parameter filter group is a simple, yet powerful feature delivered to our hands. I myself, benefited from in the following scenario.

In the company I worked for, we had a request to reroute mobile web users to the native application **only and only** when a particular query parameter with a particular value is present and when another query parameter with any value is present as well. It would enable us to personalise the experience for a given user, I cannot really tell the details now.

Although, the challange was… the domain, host, schema and path was always the same. We obviously wasn’t able to change schema nor even path, since it would influence the entire business (web+native).
Also, those links were the most common URLs (schema) used by the company. We weren’t really ready nor happy to handle all of the variations. We didn’t want to provide a generic experience, like, you’re being thrown into a main screen in 90% of times — since mobile web experience was very good. Native mobile experience was obviously excellent but still narrowed to a few features.

I want you to imagine how could you improve your user experience and business metric as well. If you had a hard time to get clues from the above article here is a short list of examples and ideas you could start implementing with fine-grained app link filters:

*   You can lower the bandwidth used — the amount of data being transferred via API calls, by both getting precise data from app links and by using those in API calls.
*   You can easily filter out the content you do not need — and narrow it to the stuff your user is really looking for.
*   You can be more specific around business metrics and analytics and how web and native apps work together.
*   You can prevent to fallback to native apps whenever they aren’t ready while attaining nice app links coverage where needed.
*   You can workaround/hackaround bad URLs schema design in your company and get what you really need for our app/user.

---

Thanks for reading, hit that clap or subscribe button hard! :)

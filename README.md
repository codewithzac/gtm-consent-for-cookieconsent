# gtm-consent-for-cookieconsent
A GTM tag template to set the standard GTM consent types using CookieConsent user preferences.

## Introduction
This template is designed to make it very easy to honour user consent preferences set via [CookieConsent by Orest Bida](https://cookieconsent.orestbida.com/) in a Google Tag Manager container.

>[!NOTE]
>Before using this template, you need to have CookieConsent up and running on your website *first*, with all of the consent categories already configured - this template doesn't load any of the JS or CSS for the consent manager!

## Download Template file
Either:
- Click the **<> Code** button above, select **Download ZIP**, or
- Go to the latest release in **Releases** on the right, then download the latest **Source code (zip)**

Extract the ZIP file, then find the `GTM Consent for CookieConsent.tpl` file within.

## Installation
1. Go to your GTM container
2. Navigate to the **Templates** menu, then click the **New** button in the Tag Templates box
3. Click the **...** menu at the top-right, then select **Import**
4. Choose the `GTM Consent for CookieConsent.tpl` file from before, then wait for it to import
5. Click the **Save** button at the top-right, then click **X** at the top-left to return to the main GTM interface

## Configure GTM container for consent
If you haven't already done so, you need to configure your GTM container for consent mode:
1. Navigate to the **Admin** tab
2. Select **Container Settings** on the right
3. Ensure **Enable consent overview** is checked
4. Click the **Save** button at the top-right if you needed to check this box

## Create Consent Events trigger
The default Consent Initialisation trigger already exists, but you also want to be able to react to consent events:
1. Navigate to the **Workspace** tab
2. Select the **Triggers** menu, then click the **New** button
3. Give the trigger a name - e.g., `Consent Events`
4. Click into **Trigger Configuration**, then select **Custom Event** from the Other section
5. If using my Wordpress plugin: enter `onFirstConsent|onChange` into the **Event name** box, then check the **Use regex matching** checkbox
6. Click the **Save** button at the top-right

## Create Consent Update trigger *(optional)*
If you have consent logging requirements, or tags that need to respond to consent changes on the page:
1. While in the **Triggers** menu, click the **New** button again
2. Give the trigger a name - e.g., `Consent Update`
3. Click into **Trigger Configuration**, then select **Custom Event** from the Other section
4. Enter `consent_update` into the **Event name** box
5. Click the **Save** button at the top-right

## Add the GTM Consent for CookieConsent tag
When adding the tag, you will need to refer to your CookieConsent configuration for the tag settings:
1. Select the **Tags** menu, then click the **New** button
2. Give the tag a name - e.g., `CookieConsent`
3. Click into Tag Configuration, then select **GTM Consent for CookieConsent** from the Custom section
4. Review the tag configuration and make sure everything matches your CookieConsent configuration - at the very least, you need to update the **Consent Type Mappings** section

Each GTM category can only be mapped to one CookieConsent category, however you can have one CookieConsent category mapped to multiple GTM categories - as shown below.

You likely want to have mappings as follows - but *don't* copy the following without checking your configuration first!:

| Google Tag Manager | CookieConsent |
| ------------------ | ------------- |
| Ad Storage         | marketing     |
| Ad User Data       | marketing     |
| Ad Personalization | marketing     |
| Analytics Storage  | analytics     |
| Strictly Necessary | necessary     |

>[!WARNING]
>If there are mismatches between the tag configuration and CookieConsent configuration, weird behaviour is almost guaranteed..

5. Click into **Triggering**, then select *BOTH* `Consent Events` and `Consent Initialisation - All Pages` (but not `Consent Update`)
6. Click the **Add** button at the top-right
7. Click the **Save** button at the top-right

## Next steps
Before publishing, test your configuration carefully to make sure everything works:
- Check each of your tags for Advanced Settings > Consent Settings, to make sure that consent is configured appropriately
- If you created the **Consent Update** trigger above, consider which tags need this added to their Triggering section, if any
- Use GTM's Preview Mode to ensure that tags are being blocked / updated according to the user consent preferences and tag settings
- Test as many different consent flows as possible: no consent set, consent choices set, consent choices updated

## Preview Mode logging
When Preview Mode is active, there are a number of comments in the log that you will see - here's what they mean:

| Comment | Explanation |
| ------- | ----------- |
| Using ( CookieConsent \| tag ) configuration, mode: ( opt-in \| opt-out ) | Default consent has been applied from either the CookieConsent configuration if available, or from the tag configuration. The mode is either opt-in (GDPR), or opt-out (CPRA). |
| Valid consent object found, contents: { object } | A valid consent object has been found in either the consent cookie or localStorage - the contents of the object are shown in the log. Consent will be applied based on this object. |
| Invalid consent object found, contents: { object } | A consent object was found in either the consent cookie or localStorage, but it wasn't valid. Default consent will be applied instead. This is very unlikely.. |
| Consent object cannot be parsed! We received: { whatever we can parse } | A consent object was found, but it couldn't be `JSON.parse()`'d. Default consent will be applied instead. This is very, *very* unlikely..! |
| Event name: ( event ), consent changes: ( n ) | The event name from whatever triggered the CookieConsent tag, commonly:<ul><li>**gtm.init_consent**: Consent Initialisation (on page load)</li><li>**onFirstConsent**: The user made their first consent choice</li><li>**onChange**: CookieConsent has detected a change of consent state from the previous state</li></ul>The number of consent changes determine whether the `consent_update` event is fired - needs to be above 0 to trigger. |

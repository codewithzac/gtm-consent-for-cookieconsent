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
The default Consent Initialisation trigger already exists, but you also need to be able to react to consent events:
1. Navigate to the **Workspace** tab
2. Select the **Triggers** menu, then click the **New** button
3. Give the trigger a name - e.g., `Consent Events`
4. Click into **Trigger Configuration**, then select **Custom Event** from the Other section
5. If you're using my [Wordpress plugin](https://github.com/codewithzac/wordpress-cookieconsent-loader) with the default configuration:
   - Enter `onFirstConsent|onChange` into the **Event name** box, then check the **Use regex matching** checkbox.
6. If you're not using my plugin, or if you've customised things:
   - Please check your CookieConsent configuration and ensure you're triggering dataLayer events for the **onFirstConsent** and **onChange** CookieConsent events - see below for what needs to be included in the configuration.
   - If you've changed the name of either of the dataLayer events, you will need to update the event name regex in the trigger configuration accordingly.
7. Click the **Save** button at the top-right

Reference: the following needs to appear in the CookieConsent configuration to trigger the dataLayer events mentioned above:
```javascript
// Trigger GTM dataLayer events when updating consent
// These are optional, but required if using the "GTM Consent for CookieConsent" GTM template
onFirstConsent: function(detail) {
 window.dataLayer = window.dataLayer || [];
 window.dataLayer.push({
   event: 'onFirstConsent',
   consentCategories: detail.cookie.categories,
   consentServices: detail.cookie.services
 });
},
onChange: function(detail) {
 window.dataLayer = window.dataLayer || [];
 window.dataLayer.push({
   event: 'onChange',
   consentCategories: detail.cookie.categories,
   consentServices: detail.cookie.services
 });
}
```
## Add the GTM Consent for CookieConsent tag
When adding the tag, you will need to refer to your CookieConsent configuration for the tag settings:
1. Select the **Tags** menu, then click the **New** button
2. Give the tag a name - e.g., `CookieConsent`
3. Click into Tag Configuration, then select **GTM Consent for CookieConsent** from the Custom section
4. Review the tag configuration and make sure everything matches your CookieConsent configuration - at the very least, you need to update the **Consent Type Mappings** section

Each GTM category can only be mapped to one CookieConsent category, however you can have one CookieConsent category mapped to multiple GTM categories - as shown below.

You likely want to have mappings similar to the following (but check your CookieConsent configuration to be sure!):

| Google Tag Manager | CookieConsent |
| ------------------ | ------------- |
| Ad Storage         | marketing     |
| Ad User Data       | marketing     |
| Ad Personalization | marketing     |
| Analytics Storage  | analytics     |
| Strictly Necessary | necessary     |

>[!WARNING]
>If there are mismatches between the tag configuration and CookieConsent configuration, weird behaviour is almost guaranteed..

5. Click into **Triggering**, then select *BOTH* `Consent Events` and `Consent Initialisation - All Pages`
6. Click the **Add** button at the top-right
7. Click the **Save** button at the top-right

## Create Consent Update trigger *(optional)*
If you have consent logging requirements, or tags that need to respond to consent changes on the page:
1. Navigate to the **Triggers** menu, click the **New** button
2. Give the trigger a name - e.g., `Consent Update`
3. Click into **Trigger Configuration**, then select **Custom Event** from the Other section
4. Enter `consent_update` into the **Event name** box
5. Click the **Save** button at the top-right

## Create Consent Type variable and All Pages - Explicit Consent trigger *(optional)*
If you have tags that need to behave differently on page load for default consent vs. consent updates (typically needed if the default consent is granted, but you only want to fire a tag if the user actually saves that preference as well) - you'll need to create a **Custom JavaScript** variable and trigger. Sadly, this doesn't work in a template, as it reads the `google_tag_data` global object.
### Variable
1. Navigate to the **Variables** menu
2. In the **User-Defined Variables** section, click the **New** button
3. Give the variable a name - e.g., `Consent Type`
4. Click into **Variable Configuration**, then select **Custom JavaScript**
5. Paste the code below, then click the **Save** button at the top-right

```javascript
function() {
  var tagData = window.google_tag_data;
  if (!tagData.ics) return;
  if (tagData.ics.usedUpdate) return 'explicit';
  if (tagData.ics.usedDefault) return 'implicit';
}
```
### Trigger
1. Navigate to the **Triggers** menu, click the **New** button
2. Give the trigger a name - e.g., `All Pages - Explicit Consent`
3. Click into **Trigger Configuration**, then select **Page View** from the Page View section
4. Choose **Some Page Views**
5. In the first drop down, choose **Consent Type**
6. In the second drop down, choose **equals**
7. In the text box, enter `explicit`
8. Click the **Save** button at the top-right

## Next steps
Before publishing, test your configuration carefully to make sure everything works:
- Check each of your tags for Advanced Settings > Consent Settings, to make sure that consent is configured appropriately
- If you created the **Consent Update** trigger above, consider which tags need this added to their Triggering section, if any
- If you created the **All Pages - Explicit Consent** trigger above, consider which tags need this added to their Triggering section, if any
- Use GTM's Preview Mode to ensure that tags are being blocked / updated according to the user consent preferences and tag settings
- Test as many different consent flows as possible: no consent set, consent choices set, consent choices updated

## Preview Mode logging
When Preview Mode is active, there are a number of comments in the log that you will see - here's what they mean:

| Comment | Explanation |
| ------- | ----------- |
| Event triggering consent evaluation: ( event ) | The event name from whatever triggered the CookieConsent tag, commonly:<ul><li>**gtm.init_consent**: Consent Initialisation (on page load)</li><li>**onFirstConsent**: The user made their first consent choice</li><li>**onChange**: CookieConsent has detected a change of consent state from the previous state</li></ul> |
| Valid consent object found, contents: { object } | A valid consent object has been found in either the consent cookie or localStorage - the contents of the object are shown in the log. |
| Invalid consent object found, contents: { object } | A consent object was found in either the consent cookie or localStorage, but it wasn't valid. Default consent will be applied instead. This is very unlikely.. |
| Consent object cannot be parsed! We received: { whatever we can parse } | A consent object was found, but it couldn't be `JSON.parse()`'d. Default consent will be applied instead. This is very, *very* unlikely..! |
| Setting default consent using ( CookieConsent \| tag ) configuration, mode is ( opt-in \| opt-out ): { object } | Default consent has been applied from either the CookieConsent configuration if available, or from the tag configuration. The mode is either opt-in (GDPR), or opt-out (CPRA). The object at the end is the consent state. |
| Setting updated consent using data from the consent object: { object } | Updated consent has been applied from the consent object. The object at the end is the consent state. |
| First consent decision - triggering consent_update event! | The `onFirstConsent` event was processed, and the `consent_update` event has been triggered. |
| Consent changed (*n* changes) - triggering consent_update event! | The `onChange` event was processed with more than 0 changes, and the `consent_update` event has been triggered. |

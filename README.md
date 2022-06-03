# QPhoneInput - Phone number validator / formatting

Supports international phone numbers with auto formatting as you type that adapts to international phone validation and number formatting rules managed by Google. 

Uses [`libphonenumber-js`](https://catamphetamine.gitlab.io/libphonenumber-js/) to minimise bundle download size.  

## Demo

[Fork and run demo on StackBlitz ⚡️](https://stackblitz.com/fork/github/tohagan/quasar-q-phone-input)

## TODO
- Add a `rules` property to replace hard coded validation rules in this demo.
- Improve smarts related to auto switching country (works but could be improved)
- Lazy load `libphonenumber-js` and property to select the bundle size (min, max or mobile).
- Convert to Quasar Extension

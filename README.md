# converse-tel

Type a phone number into [Converse.js](https://conversejs.org)'s add-contact
form; get the right gateway JID.

Without this plugin, adding an SMS contact through an XMPP↔SMS gateway
(JMP/Cheogram, or a self-hosted equivalent) means knowing the gateway's JID
convention and typing `+16155551212@gateway.example` by hand. With it, you type
`(615) 555-1212` and the plugin asks your gateway to translate.

## How it works

Translation uses [XEP-0100](https://xmpp.org/extensions/xep-0100.html)'s
address lookup protocol (`jabber:iq:gateway`): the client sends the number to
the gateway as a `<prompt>`, the gateway answers with the `<jid>` it maps to.
The number is sent exactly as typed — normalization is the gateway's job.

**No gateway is hardcoded.** Candidates are, in order:

1. The `tel_gateways` setting, if non-empty.
2. Auto-discovery: domain-only JIDs in your roster. A gateway subscription
   (e.g. `cheogram.com`) lives in the roster as a bare domain, which nothing
   else does.

The first gateway that answers successfully is cached and asked first next
time. Change providers — or move to a self-hosted gateway — and nothing here
needs to change.

Input that isn't phone-shaped (a real JID, a bare username) bypasses the
plugin entirely, so normal add-contact behavior is untouched.

Converse has no official hook for the add-contact flow, so the plugin patches
`addContactFromForm` on the `converse-add-contact-modal` custom element
prototype. Any plugin failure falls through to Converse's stock handling —
a bug here can degrade to the old behavior, never break the form. Verified
against Converse.js v14.

## Usage

Load after the Converse bundle (both as ES modules), then whitelist:

```html
<script type="module" src="/dist/converse.min.js"></script>
<script type="module" src="/converse-tel.js"></script>
```

```js
converse.initialize({
    // ...
    whitelisted_plugins: ['converse-tel'],
});
```

## Settings

| Setting        | Default | Meaning                                                          |
| -------------- | ------- | ---------------------------------------------------------------- |
| `tel_gateways` | `[]`    | Gateway JIDs to query, in order. Empty = auto-discover (roster). |
| `tel_debug`    | `false` | Log lookups and failures to the console.                         |

## License

MIT

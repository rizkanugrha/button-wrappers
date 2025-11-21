# Baileys Interactive Message Wrapper
Support Baileys 7.0.0

A robust utility wrapper for [@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys) designed to simplify the creation, validation, and sending of **Interactive Messages** (Native Flow, Buttons, Lists).

This library solves the common issue of buttons not rendering by automatically handling the complex binary node injection (`biz` tags) required by modern WhatsApp versions.

## 🚀 Features

- **Simplified Syntax**: Send buttons using a clean JSON object structure via `sendButtons`.
- **Strict Validation**: Built-in validation system that checks your JSON payloads against required fields before sending, preventing silent failures.
- **Native Flow Support**: Full support for `cta_url`, `cta_copy`, `single_select` (List Messages), `cta_call`, and more.
- **Auto-Legacy Conversion**: Automatically converts simple `id`/`text` button objects into the modern `quick_reply` format required by the protocol.
- **Binary Node Handling**: Automatically injects `biz`, `bot`, and `interactive` nodes into the binary protocol to ensure rendering on the client side.

## 📂 Installation

Since this is a custom utility module, follow these steps:

1.  Ensure you have `@whiskeysockets/baileys` installed in your project.
2.  Create a directory, e.g., `src/utils/interactive`.
3.  Copy the following files into that directory:
    - `index.js`
    - `send.js`
    - `validators.js`
    - `builders.js`
    - `internal.js`

## 🛠️ Usage

### 1. Import the Library

```javascript
import { sendButtons, sendInteractiveMessage } from './src/utils/interactive/index.js';
// 'sock' refers to your main Baileys socket connection
```
### 2. Old Buttons (sendButtons)

```javascript
await sendButtons(sock, jid, {
            text: 'Coba beton',
            footer: 'anu',
            buttons: [
                { id: `${prefix}menu`, text: 'Menu' },
                { id: `${prefix}infoserver`, text: 'Info Server' }
            ]
        });
```

### 3. Example use of several types of buttons (InteractiveMessage)
Use sendInteractiveMessage for scenarios like Quick Replies, URL buttons, or Copy Code buttons. The library automatically handles the JSON serialization for buttonParamsJson.

```javascript
const jid = '1234567890@s.whatsapp.net';

try {
    await sendInteractiveMessage(sock, jid, {
        title: 'Account Verification',
        text: 'Please select an option to proceed with your verification.',
        footer: 'Security System',
        interactiveButtons: [
            // 1. Standard Quick Reply (Legacy format supported)
            { id: 'verify_now', text: 'Verify Now' },

            // 2. Call-To-Action: URL
            {
                name: 'cta_url',
                buttonParamsJson: JSON.stringify({
                    display_text: 'Visit Website',
                    url: '[https://example.com/verify](https://example.com/verify)'
                })
            },

            // 3. Call-To-Action: Copy Code
            {
                name: 'cta_copy',
                buttonParamsJson: JSON.stringify({
                    display_text: 'Copy OTP',
                    copy_code: '123-456'
                })
            }
        ]
    });
    console.log('Buttons sent successfully');
} catch (error) {
    console.error('Failed to send buttons:', error.message);
}
```

### 4. List Messages (single_select)
To send a "List Message" (Radio button menu), you must use the single_select type.

```javascript
await sendInteractiveMessage(sock, jid, {
    text: 'Please select a product category',
    footer: 'Catalog',
    interactiveButtons: [
        {
            name: 'single_select',
            buttonParamsJson: JSON.stringify({
                title: 'Open Menu', // Text on the button
                sections: [
                    {
                        title: 'Electronics',
                        rows: [
                            { header: 'Laptops', title: 'Gaming & Work', description: 'High performance', id: 'cat_laptop' },
                            { title: 'Smartphones', id: 'cat_phone' }
                        ]
                    },
                    {
                        title: 'Clothing',
                        rows: [
                            { title: 'Men', id: 'cat_men' },
                            { title: 'Women', id: 'cat_women' }
                        ]
                    }
                ]
            })
        }
    ]
});
```

### 5. Send Locations

```javascript
await sendInteractiveMessage(sock, jid, {
  text: 'Silakan bagikan lokasi Anda',
  interactiveButtons: [
    { 
      name: 'send_location', 
      buttonParamsJson: JSON.stringify({ 
        display_text: 'Bagikan Lokasi' 
      }) 
    }
  ]
});
```

### 6. Advanced Payload 
For full control over the message structure (headers, detailed body, raw native flow), use the lower-level sendInteractiveMessage function.

```javascript
const rawContent = {
    body: { text: 'Advanced interactive message' },
    footer: { text: 'Footer text' },
    header: { title: 'Header Title', subtitle: 'Subtitle' },
    interactiveMessage: {
        nativeFlowMessage: {
            buttons: [
                {
                    name: 'cta_call',
                    buttonParamsJson: JSON.stringify({
                        display_text: 'Call Support',
                        phone_number: '+1234567890'
                    })
                }
            ]
        }
    }
};

await sendInteractiveMessage(sock, jid, rawContent);
```

### 🧩 Supported Button Types
The library validates specific fields for the following button types:

| Button Name | Required Fields in `buttonParamsJson` | Usage |
| :--- | :--- | :--- |
| `quick_reply` | `display_text`, `id` | Standard reply button. |
| `cta_url` | `display_text`, `url` | Opens a web page. |
| `cta_copy` | `display_text`, `copy_code` | Copies text to clipboard. |
| `cta_call` | `display_text`, `phone_number` | Initiates a phone call. |
| `single_select` | `title`, `sections` | Opens a list/menu. |
| `address_message` | `display_text` | User sends their address. |
| `send_location` | `display_text` | User sends live location. |
| `mpm` | `product_id` | Multi-Product Message. |


### ⚠️ Error Handling
The library exports a custom error class InteractiveValidationError. It provides detailed context on exactly which button or field failed validation.
```javascript
import { InteractiveValidationError } from './src/utils/interactive/index.js';

try {
    await sendButtons(sock, jid, invalidPayload);
} catch (err) {
    if (err instanceof InteractiveValidationError) {
        console.error('Validation Failed!');
        console.error(err.formatDetailed());
        // Example Output:
        // [InteractiveValidationError] Buttons payload invalid
        // Errors:
        //   - button[0] (cta_url) missing required field 'url'
    }
}
```

📜 Architecture

send.js: Main logic. Calls generateWAMessageFromContent and appends the additionalNodes (binary tags) needed for buttons to work.

validators.js: Contains REQUIRED_FIELDS_MAP and logic to parse JSON strings inside buttons to ensure they are valid before sending.

internal.js: Determines if the message is a list, buttons, or native_flow and constructs the corresponding biz tag structure.

builders.js: Helper utilities to map simple array inputs into the complex object structure WhatsApp expects.

🙏 Thanks To

[@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys)

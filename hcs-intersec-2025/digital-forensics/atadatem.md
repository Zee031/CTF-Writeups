---
description: 'Challenge author: Bim'
---

# atadatem

The challenge first gives an image file with the name hcs.png

<figure><img src="../../.gitbook/assets/atadatem_hcs.png.png" alt=""><figcaption></figcaption></figure>

Nothing special here, so I went ahead and checked the metadata with exiftool

<figure><img src="../../.gitbook/assets/atadatem_metadata.png" alt=""><figcaption></figcaption></figure>

first, it was the comment and description that stood out to me as it seemed like base64 encoded strings, then the title itself seemed to suggest the actual order of the base64 encoded string

<figure><img src="../../.gitbook/assets/atadatem_decode.png" alt=""><figcaption></figcaption></figure>

Decoding the string gave an reversed URL, so I used ChatGPT to reverse it back and got the URL\
https://pastebin.com/quC66zAQ

<figure><img src="../../.gitbook/assets/atadatem_pastebin.png" alt=""><figcaption></figcaption></figure>

The last string to decode

<figure><img src="../../.gitbook/assets/atadatem_flag.png" alt=""><figcaption></figcaption></figure>

Flag: HCS{metadata\_can\_be\_useful\_somewhere\_somehow\_sometime}

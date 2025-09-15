---
description: 'Challenge author: Bim'
---

# atadatem

The challenge first gives an image file with the name hcs.png

<figure><img src="../../.gitbook/assets/{C4F97417-24CB-4CBC-A800-1EFB73764E8F}.png" alt=""><figcaption></figcaption></figure>

Nothing special here, so I went ahead and checked the metadata with exiftool

<figure><img src="../../.gitbook/assets/{D795D0D4-D9F2-455E-B6A6-EF7951346503}.png" alt=""><figcaption></figcaption></figure>

first, it was the comment and description that stood out to me as it seemed like base64 encoded strings, then the title itself seemed to suggest the actual order of the base64 encoded string

<figure><img src="../../.gitbook/assets/{CC5A0162-9A38-4E42-A1D4-1B22549AA9FE}.png" alt=""><figcaption></figcaption></figure>

Decoding the string gave an reversed URL, so I used ChatGPT to reverse it back and got the URL\
https://pastebin.com/quC66zAQ

<figure><img src="../../.gitbook/assets/{C2517D91-400B-4C29-B5DE-3F7602EFA333}.png" alt=""><figcaption></figcaption></figure>

The last string to decode

<figure><img src="../../.gitbook/assets/{04DFE832-FE99-456A-AB8A-30DFAB12D9EB}.png" alt=""><figcaption></figcaption></figure>

Flag: HCS{metadata\_can\_be\_useful\_somewhere\_somehow\_sometime}

---
description: 'Challenge author: Vaa'
---

# based

The challenge first gave a zip file that contains a text file with the name of out.txt

<div data-full-width="true"><figure><img src="../../.gitbook/assets/based_out.txt.png" alt=""><figcaption></figcaption></figure></div>

Looked like basic base64 encoded string, so I went ahead and used cyberchef for this job

<figure><img src="../../.gitbook/assets/based_decode1.png" alt=""><figcaption></figcaption></figure>

After one decoding, it still resulted in a base64 string, so I assumed that it was encoded with base64 multiple times and decoded it repeatedly until I found the flag

<figure><img src="../../.gitbook/assets/based_decode2.png" alt=""><figcaption></figcaption></figure>

Flag: HCS{ju5t\_b4s3\_64\_l0l}

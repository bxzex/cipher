# Cipher

AES, GCM, SHA-256 and HMAC written from scratch in JavaScript, so you can see what each one is doing.

https://bxzex.github.io/cipher/

Pick an AES round and you can see the state after each step, with the bytes that changed marked. There's also a diffusion view where one flipped bit spreads through the state within two rounds. On the SHA-256 side you get an avalanche grid and the message schedule.

The Verify tab checks everything against the published NIST and RFC test vectors, then runs about a thousand random cases against the browser's WebCrypto. Everything matches. Writing the vector table, I copied one HMAC value with a character missing, and the test caught it straight away.

This is for learning, not for real use. It isn't constant time.

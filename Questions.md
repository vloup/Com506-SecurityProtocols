This is a list of questions I received.

### Q1
Q: In the second paragraph you mention that the E&M construction the MAC is not required to provide confidentiality. Well, how would a MAC leak part of the plaintext in the output? That happens if no encryption is done.. when you send M || MAC(M). But this is not the case, since the message is sent encrypted.

A: In the first part of the presentation (based on probabilistic encryption), the authors have taken the definition of the MAC from its required properties, nothing more.
A MAC should provide:
* Authenticity
* Integrity

Nowhere, it is mentioned that a MAC needs to provide confidentiality, that's why E&M is considered bad.
If you look at the counterexample in the BN paper, you will see that they construct a MAC having those two properties, but leaking a bit of the plaintext.
They just prepend the first bit of the plaintext to a PRF-based MAC, and this breaks confidentiality of the encryption scheme.
During the second part of the presentation, the authors changed the definition of the MAC and took it as a PRF, something that totally make more sense.

### Q2
Q: In paragraph 3: you say that the approach does not apply to some real cryptographic algorithms that are based on a nonce or on an IV. What does it mean? The same for the thing of the associated data. What point are you trying to make? Do you mean that a new approach is necessary in order to consider the "real-world" techniques, such as GCM? And this new approach would be the one that was done in 2014 by those 2 guys and that is described in paragraph 4 of the report? 

A: Well, how would you take into account the nonce/IV or the associated data in the classic EtM?
Nowhere it is mentioned where and how you should authenticate those information.
If you do not authenticate the associated data, you can make trivial forgeries (just change the AD, and you are done).

So, most definitely the approach of probabilistic encryption is not suitable since you have extra data which you do not know how to authenticate.
This problem comes from the fact that building probabilistic encryption (same messages get encrypted differently) out of deterministic encryption (same messages get encrypted to the same ciphertext) is done by providing randomness to the encryption schemes via the IV or the nonce.
Associated data is just a plus for flexibility.
It helps to authenticate stuffs you cannot encrypt (say, an IP packet header, otherwise it wouldn't get routed properly).
That's exactly the path the two authors went in 2014, asking themselves how to do generic composition when you now have the associated data and the IV/Nonce.

### Q3
Q: Paragraph 4: when the security of nAE is defined, there is a point saying: "The output of the encryption is indistinguishable from random strings in a chosen plaintext attack, to which the adversary must not repeat nonces. " --> what does "to which the adversary must not repeat nonces" mean? Does it mean that an adversary cannot use the same nonce multiple times when choosing the plaintext to give to the oracle? 

A: YES, you do not repeat nonces, ever. Not even the adversary is allowed to do it.
Give two oracles to the adversary, one is a real encryption oracle, the other one just outputs random bitstrings.
He can submit as many plaintext messages as he wish (he chooses them, hence the chosen plaintext attack).
Ask the adversary to tell which one is encrypting, and which one is random, on the only condition he should not repeat nonces when using the oracles.
Our security notion is that the adversary should not be able to differentiate the two oracles.

### Q4
Q: Paragraph 4.1 (end): you mention that there are 160 schemes to evaluate. On the slides you say that there are 120. Which is the correct information? How to you get that number? I am trying to get it manually, but I cannot find a good way to enumerate all the schemes.

A: Errata, 120 is a typo :-)
160 = 2⁶ + 2⁵ + 2⁶.
They classified each A-schemes:
* A1: iv = F(N | \_, A | \_, M | \_), T = F(N | \_, A | \_, M | \_) and return C || T : 2⁶ candidates
* A2: iv = F(N | \_, A | \_, M | \_), T = F(N | \_, A | \_, C) and return C : 2⁵ candidates.
* A3: iv = F(N | \_, A | \_, M | \_), T = F(N | \_, A | \_, M | \_) and return C = E(iv, M || T) : 2⁶ candidates

### Q5
Q: Slide 19: here you show the constructions from A1 to A8. What is that "epsilon" that gives the stuff to the cipher?

A: Encryption with key K. I admit I used a simple letter 'E' for that presentation. Many do write encryption with the epsilon letter.

### Q6
Q: How do you assess security of those schemes? Why is there a smaller number of secure solutions in the technique used in paragraph 4.3 (only 3 favored)? Shouldn't the IV and the nonce be equivalent? 

A: First they reduced the schemes with automatic counterexample finding with easy attacks.
Then they went to prove by hand by bounding probabilities for forgeries and message recovery and compared this to a PRF.
There are only three favored because we also have way less candidates (only 20), since we directly use the Nonce in the nonce-based symmetric scheme and not use F to create the IV to make it random.
Also, nonce and IV are definitely not equivalent.

Do not get confused.
* IV: random initialization vector
* Nonce: unique initialization vector. It can be a counter.

### Q7
Q: Slide 24: what is the problem even if EtM is poorly defined? The other stuff (GCM, CCM, EAX) seem to work well... although in paragraph 2 you mentioned that only EtM is secure: is the problem related to this?

A: If you don't know crypto and you implement EtM from this standard, you're surely gonna have a bad implementation.
It's a matter of understanding the crypto results and not cooking our own crypto.
The problem comes from their creation of the SV value.
It's just unclear what they want it to be (Nonce or IV) and how tight the security of this value should be.
It goes into domino effect and make them try to apply a probabilistic encryption construction to a IV/Nonce (or whatever SV is)-based scheme, which it definitely not the result from the first paper.

I try to already introduce the problem in slide 11.
Also, GCM or CCM is not generic composition schemes, it's a dedicated mode of operation.
EAX is scheme B1 (so it's proven secure as long as the encryption and authentication functions are secure!).

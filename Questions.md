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
During the second part of the presentation, the authors changed the definition of the MAC and took it as a PRF, something that totally makes more sense.

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

### Q8
Q: Is there a real advantage to favor (or not favor) a truncatable scheme?

A: Not really. All schemes in the A/B/N class have been shown to be secure as long as their underlying primitives are also secure.
You might say that you gain flexibility when using a truncatable scheme, since you can decide to have a shorter tag.
Still, you want to have a fixed size tag, otherwise you could just make a trivial forgery by removing the last bit of the ciphertext and pretending it is shorter.
Also, a tag that is too short is unsecure as well.

On the opposite, untruncatable schemes just get rid of that property, by "hiding" the tag behind encryption.

### Q9
Q: Which scheme A/B or N-schemes would you recommend if you want to create a new protocol?

A: It depends, but of course you need to pick the favored schemes that are proven secure, not those with a weaker or unknown security bound.
There are the already known schemes that might get favored (because they have been more studied in the past).
For example, A4 and B1 were already known as the SIV and EAX mode of operation.

Secondly, it depends to what you have in hand in term of libraries or code.
If you can only construct B-schemes, no need to go an try to get an N-scheme implemented since you are likely to be in troubles.

Then, if you look closely how many strMAC are being used in all schemes (B or N schemes if you do the three-xor construction), you can see that we just strMAC each input parameter.
So no matter what, you will perform 3 strMAC with all B/N schemes.
After that, it is just a matter of xoring bitstrings together and use the encryption function once, this is not a costly operation.
So for speed, the real only advantage is in finding parallelism.

The N schemes may get favored in this case because they directly use the nonce in encryption.
Look at N1 and N2, with N1 you can encrypt in parallel of creating the tag, and N2 can decrypt in parallel of the verification the tag.
As you see, it is impossible to have a way to do both encryption and decryption in parallel, you have to favor one for the speed.
That's just a tradeoff you have to make.
Also, if you want to go full crazy on parallelism, there are some interesting nonce-based modes of operation that can get trivially parallelized.
The CTR mode of operation is an example, since it allows you to encrypt and decrypt each block concurrently.

But, out of honesty, GCM is also crazily fast too, and not covered by this presentation.
For people that submitted new AEAD (Authenticated Encryption with Associated Data) cipher to the CAESAR competition, you have to demonstrate why you should prefer your submission compared to AES-GCM, that's just how fast it is.
But hey, CAESAR is still ongoing, so there are no winner yet.
In the end, you should not hesitate to look outside the constructions presented during this presentation :-)

### Q10
Q: When you define the reject symbol in the case of nAE, you simply define it for decryption and not for encryption. But when you present the required properties of nAE you write E(K, N, A, M) = C != \bot. I find it easy to imagine that this defines as: when the encryption doesn't result in an error, but I would rather get a confirmation of that.

A: That's a good question you have and you are correct.
I took a shortcut on that one since the only reason to have an error during encryption is because you provided incorrect parameters (formally in the paper: in the case of a parameter that does not lie in its defined domain).
So, you should hardly never get \bot in practice.
This definition also applies to IV-based encryption schemes.

I've written it in the properties since you do not decrypt \bot (again, it's not in the ciphertext domain, so there is no meaning to "decrypt" it) and briefly mentioned it during my presentation.

All in all, the most important thing is not to ignore the \bot appearing in decryption.
That one is really important since it tells us that:
* All parameters are within their defined domain.
* Decryption did occur without making any error (for example no padding error).
* And that the authentication + integrity of the message (and the other parameters) got validated.

### Q11
Q: Can you give a vague idea about how the proofs are done?

A: In this paper, the proof is done in a rather classical way.
First, they decided to define which type of attack they will study (forgery attempt).
Secondly, they did define the power of an adversary (what tools he has in hands to attack our scheme, usually an encryption and decryption oracles) and its goal (create a new fresh ciphertext that is valid).
Then, the proof goes by letting the adversary "play" with what we provided him.

From there, we can estimate the probability of success for our adversary to win, and see if it can be better than just a random guess.
If the adversary cannot do any better than a random guess, our scheme is proven secure.
It's doable since you assume that the vecMAC function behaves like PRF.
The result of the probabilities can be found in Figure 9 of the paper, with q\_d being the number of queries to the decryption oracle.

The proof to show that a scheme is not secure is done by counter example.
You have a list of them on page 24 up to 26.
It's just a method to follow when given oracles (both encryption, decryption).
The one I showed during the presentation is Atk-5.

Keep in mind that the important thing to know is the results of this paper.
The detail of the proof can be mostly omitted, but having a vague idea how they did it is still a good thing.

### Q12
Q: Can you elaborate on malleability in the case of probabilistic encryption? (question slightly altered)

A: Malleability is the property to have two distinct ciphertexts that gets decrypted to the same plaintext.
You can easily create such ciphers by just appending a sequence of random bits at the end of the ciphertext after encryption, and truncating them right before decryption.
A curious user would just change any bits and still get the same plaintext.
Usually, as a rule of thumb, malleability happens when the ciphertext is longer than the plaintext (pigeonhole principle).
If not, your encryption/decryption scheme is not bijective, and you might end up having problems encrypting some messages that is not mapped to anything in the ciphertext domain.

So, given a valid ciphertext message, if you can find a second message that gets decrypted to the same plaintext, you are able to break E&M and MtE.
The issue comes from the fact that we authenticate the plaintext, not the ciphertext.
With these constructions, you inherit the same property as the encryption function (malleability) and end up with forgery attacks.
EtM does authenticate the ciphertext, so you do not suffer from this issue at all.

In the end, pAE is what you try to get by creating EtM, MtE and E&M schemes.
You want to avoid any forgeries since you add authentication on top of the encryption scheme.
We just show that, by looking at the encryption function as a probabilistic one, malleability kills the property of being secure against forged ciphertext in the MtE and E&M construction, but is still secure when being used with the EtM construction.

### Q13
Q: In the first part of the presentation, do we not simply care about the plaintext to be authenticated?

A: In the case of a malleable cipher, authenticating the plaintext only is not sufficient.
If you do so, you do not know which exact ciphertext got your authenticated plaintext.

This subtle detail allows enough room for an adversary to take your original message, change the ciphertext but still have a valid plaintext/tag, and replay it later.
The recipient of the message will be fooled even if it asks for fresh/unseen ciphertext/tag tuple.

Practically, as an example, you can have initial encrypted+authenticated message that says "sell half of my stocks from company whatever".
An adversary intercepting the message can then generate a second fresh new message and sell the other half of your stocks.

Turns out that, in the case of probabilistic encryption, authenticating what you send is better than authenticating what you mean.
Later in the presentation, we require the nAE to have correctness and tinyness. The properties do remove malleability.

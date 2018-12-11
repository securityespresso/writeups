---
title: Birdman's Data
contest: OtterCTF 2018
authors: Lucian Nitescu
layout: writeup
---


## Description:

We recorded some of BirdMan's networking, but a part of it (the important part) got scrambled.

[Download](https://nitesculucian.github.io/uploads/otter3/Birdmans_data.pcap)

## Stats:

150 points / 100 solvers

## Solution:

  
While looking throughout the provided ```.pcap``` I discovered the following issue:

Someone used [http://www.txtwizard.net/crypto](https://www.google.com/url?q=http://www.txtwizard.net/crypto&sa=D&ust=1544488687153000) application to encrypt a message that can now be retrieved. The web application server uses ```HTTP``` instead of ```HTTPS```, therefore, all network traffic can be intercepted by an attacker.

I decided to retrieve the JSON files used in the encryption process using the ```Network Miner``` tool.

![](https://nitesculucian.github.io/uploads/otter3/image1.png)

Key A:

```json

{"key":"sEhrZxQpnNnINixu3KQ1Tg=="}

```

Key B:

```json

{"key":"XfCtxvD1yFZbxQ/+ULhAcA=="}

```

The actual Cipher Text (output of encryption):

```json

{"cipherText":"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"}

```

Knowing that we have two keys (Called A and B) and a ciphertext, we can determine that it is a ```CBC``` based algorithm and we must identify from A and B which is the ```Key``` and the ```IV```. I used Wireshark for further research and I discovered that the intercepted (victim), used a POST requests to ```/crypto/AES/encrypt/CBC/PKCSS```:

![](https://nitesculucian.github.io/uploads/otter3/image2.png)

And the victim used the ```Key``` and the ```IV``` as such:

![](https://nitesculucian.github.io/uploads/otter3/image5.png)

I have used the same tool as the victim to decrypt the ciphertext:

![](https://nitesculucian.github.io/uploads/otter3/image4.png)

Output:

```

Chance Something's wrong, I can feel it (Six minutes, Slim Shady, you're on) Just a feeling I've got, like something's about

To happen, but I don't know what If that means, what I think it means, we're in trouble, big trouble, And if he is as bananas as you say, I'm not taking any chances You were just what the doctor ordered I'm beginning to

Feel like a Rap God, Rap God All my people from the front to the back nod, back nod Now who thinks their arms are long

{

Enough to slap box, slap box? They said I rap like a robot, so call

me Rapbot But for me to rap like a computer must be

in my genes I got a laptop in my back pocket My pen'll go off when I half-c*** it Got a fat knot from that rap profit Made a living and a killing off it Ever since Bill Clinton was still in office With Monica Lewinsky feeling on his

Nut-sack I'm an MC still as honest But as rude and indecent as all hell syllables, killaholic (Kill 'em all with) This slickety, gibbedy, hibbedy hip hop You don't really wanna get into a pissing match with this rappidy rap Packing a Mac in the back of the Ac, pack backpack rap, yep, yackidy-yac The

exact same time I attempt these lyrical acrobat stunts while I'm practicing That I'll still be able to break a

Motherf***in' table Over the back of a couple of

_

Faggots and crack it in half

Only

Realized it was ironic I was signed to Aftermath after the fact How could I not blow? All I do is drop F-bombs, feel my wrath of attack Rappers are having a rough time period, here's a Maxipad It's actually disastrously bad For the wack while I'm masterfully constructing this masterpiece as I'm beginning to feel

_

Like a Rap God, Rap God All my people from the front to the back nod, back nod Now who thinks their arms are long enough to slap box, slap box? Let me show you maintaining this s*** ain't that hard, that hard Everybody want the key and the secret to rap

immortality like I have got Well, to be truthful the blueprint's simply rage and youthful exuberance Everybody loves to root

for a nuisance Hit the

Earth like an asteroid, did nothing but shoot for the moon since MC's

_

get taken to school with this music Cause I use it as a vehicle to bust a rhyme Now I lead a new school full of students Me? I'm a product of Rakim, Lakim Shabazz, 2Pac N- -W.A, Cube, hey, Doc, Ren, Yella,

Eazy, thank you, they got Slim Inspired

enough to one day grow up, blow up and be in a position To meet Run DMC and induct them into the motherf***in' Rock n' Roll Hall of Fame Even though I walk in the church and burst in a ball of flames Only Hall of Fame I be inducted in is the alcohol of fame On the wall of shame You fags think it's all a game 'til I walk a flock of flames Off of planking, tell me what in the f*** are you thinking? Little gay looking boy So gay I can barely say it with a straight face looking boy You witnessing a massacre Like you watching a church gathering take place looking boy Oy vey, that boy's gay, that's all they say looking boy You get a thumbs up, pat on the back And a way to go from your label everyday looking boy Hey, looking boy, what you say looking boy? I got a "hell yeah" from Dre looking boy I'mma work for everything I have Never ask nobody for s***, get outta my face looking boy Basically boy you're never gonna be capable To keep up with the same pace looking boy 'Cause I'm beginning to feel like a Rap God, Rap God All my people from the front to the back nod, back nod The way I'm racing around the track, call me Nascar, Nascar Dale Earnhardt of the trailer park, the White Trash God Kneel before General

zorb

}

```

If we take a look at the above text paragraph and we take only the first character of each line, we obtain the following:

![](https://nitesculucian.github.io/uploads/otter3/image3.png)

```

CTF{EmiNeM_FOR_LifE_gEez}

```

![](https://nitesculucian.github.io/uploads/otter3/image6.png)

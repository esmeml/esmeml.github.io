---
layout: post
title: A Gift for a Friend's Den
date: 2018-10-21 16:00:00 -06:00
license: cc0
published: False
categories:
- Fun
tags:
- NeuralNetworks
- TensorFlow
- Robots
---
Once again it has been quite a while since I updated this blog. Strangely, the
last time it happened was also in October. There must be something in the air
around this time of year. Like cold. There's cold in the air.

Anyways this post is due to a gift I am working on for [UrbanFriendDen][1], aka
Ruben. While I don't talk to Ruben often, he is an absurdly talented writer and
all around Good Dude(TM). He [recently tweeted][2] that tomorrow is his
birthday. Technically, he asked people to overthrow the American Imperialist
Regime, but I am very tired and still in bed at 4PM, so I proposed something a
little different.

The following blog post will detail two separate projects, one of which Ruben
already knows about and the other he does not. In the first project, we will
create a corpus of his [DICKHARDBOILED Series][3] and the
[Communist Manifesto][4]. We will then train a [TensorFlow][5] network on this
corpus following the [wonderful article][6] written by Max Woolf. While I would
normally like to roll my own network, that takes an absurd amount of time and
Ruben's birthday is tomorrow.

In the second project, we will using the same training guide as used for project
one. The difference is that we will first be scraping Ruben's [wordpress site]
and downloading all of his writings. Once done, they will be assembled into a
corpus which the network will be trained on. We should end up with an
interesting little network that sounds quite a bit like his fantastic writing
style.

As a side note in which I gush, I'm such a huge fan of Ruben that I've planned
on commissioning him to write corpus texts to train another neural network I'm
building as a hobby. If anyone reading this remembers the original version of
Seara, a sassy little helper robot/chat bot that did moderation on the Collegi
Pixelmon server, this would be version 2.0 of her. Her new model will have a
generative text backend that responds when her retrieval model doesn't have
a programmed solution, but to do so it requires a LOT of text. I'm hoping to be
able to commission Ruben to write me text in her voice. I reached out to him
once before, but alas, money is tight, and artists should always be compensated.

Anyways, off we go to work.

__Important Consideration__
While all of my writing is available under CC0/Public Domain (See the "No Rights
Reserved" link in the sidebar), Ruben's writing remains HIS OWN. Nothing in this
article should be construed as sub-licensing his work or making it available in
the public domain. I have asked for permission to release the training data,
but even if the data is released it still remains his intellectual property.
Don't be an ass, credit artists for their work.

__Additional Update__
While I was writing this, Ruben granted permission for me to upload the data
sets so that they are publicly available. I will be posting the link to the
github repository once I've finished this article. These files are made
available with permission from Ruben. If you attempt to use them to produce
any derivative works that you intend to publish, please contact him before
doing so. He's fairly chill, but again this is his intellectual property.

## Project One - Workers of Neo Noir Dark Noir City... UNITE!! ##
This one is actually fairly simple for us to get started on. I created a new
directory inside my development workspace named "Ruben_Present", and opened
that up inside the [Atom][7] editor. Once I had defined it as a project
directory, I created a subdirectory named "Project_1."

Then I copied all of the text from Project Gutenberg's copy of the Communist
Manifesto that was linked to above and saved it to "manifesto.txt." Once done,
we just had to go through and remove extraneous information. Everything inside
the file will be used to train the neural network, and it is incapable of
distinguishing between headers, sub-headers, or body text. Since we write
headers differently, they need to go. Formatting doesn't honestly matter.

During that stage I also updated anything the spellchecker flagged to modern
day spelling. Part of training a neural network is related to vectorization,
and different spellings of the same word would be treated as different words,
introducing unneeded complexity into the model. If there was no equivalent word,
I left it alone.

After doing this, I took a look at the total size of manifesto.txt. We had a
72KB file, which meant that I would need at least 72KB of Ruben's writing to
be able to cross balance the two.

First I pulled down the [original story][3]. I applied the same process that
was applied to the manifesto, checking for text marked by the spell checker,
removing extraneous formatting (legs woman's atypical font, for example), and
reducing everything to 80 characters per line. (I feel it is worth noting that
during this process I attempted to smoke two vaporizers simultaneously -- it
seemed appropriate for the subject matter -- but my significant other frowned at
me until I went back to just a single one. I still made sure that smoke was
leaving my face at the maximum possible velocity. It's worth noting that I
choked at "inject an epi-pen of smoke directly into my lungs." I really need
to quit.)

After processing the original story, I took a look at the file size. We only
had 10KB of data versus the 72KB of the manifesto. Luckily, Ruben has written
more than one story for Mr. Hardboiled. Puffing furiously on my nicotine
delivery device, Noir-inspired inspiration inspired me. I smoked the-- sorry,
I added the additional stories. You can find the first additional story
[here][8]. The second one is located [here][9].

After working on this, I once again checked our total data quantity. We had
22KB of Dick Hardboiled. 50KB short of what we would need to have a
semi-balanced model. I spent a long time thinking about what to do. I could
scrape some of Ruben's other writing, but that would infringe on surprise number
two. I decided that I would scrape Ruben's twitter and put a piece of himself
into the engine. The easiest way it seemed to do this was a python library
called [twitter-scraper][10]. So I needed to create a throwaway python
environment, install twitter scraper, and then pull down as many of Ruben's
tweets as I could. This is somewhat of a process so you can watch an ASCIInema
of the process below. Remember, you can directly copy from this playing video
if you wanted to do this yourself. You can find my comments in the video stream.

This is about a 12 minute video, and isn't really needed unless you want to see
how I obtained the twitter data.

[![asciicast](https://asciinema.org/a/gpQn8si0ctr59fqrtWTKkbz9j.png)](https://asciinema.org/a/gpQn8si0ctr59fqrtWTKkbz9j)

The initial scrape yielded approximately 57KB of data. So I got to be a bit
selective with what I kept. I spent a period of time processing the tweets by
hand according to the same rules as listed above.

While doing this however, I realized that I also had scraped retweets. Part of
the idea in doing this was to ensure that Ruben's voice would stay at the for
front. Also, you can tell this article was a stream of work from beginning to
end and I am so sorry. Anyways, after looking at the data I had collected,
I decided to pull down some additional stories from Ruben's website to get to
the amount of data I needed. I discarded the twitter scrape.

The most appropriate story to scrape to flesh out the data requirement was
"A Cyber Punk." To ensure hardboiled wouldn't be drowned out, I duplicated the
text twice within the hardboiled file. This may come back to bite me. The end
result is that I needed approximately 30KB.

I ended up pulling down parts 1-6 to get a sufficient quantity of data. I then
began the process of formatting the text to 80 characters per line, correcting
any spelling "mistakes" and generally removing extraneous formatting. Around
this time I decided I would actually end up making two models for this first
project.

The first model will be JUST the manifesto and Hardboiled, which will lead to
a... neo-noir inspired Karl Marx, and the second will be the manifesto with
hardboiled and A Cyberpunk. This should give us half/half karl marx and Ruben.

Both models will be made available and I will post the resulting 1000 words from
each model in this article.

I'm also out of nicotine at this point. My blood is more blood than nicotine.

It's fuckin terrible.

(As a fun side note, I'm processing the text as I write this, which means this
article also has a bizarre number of stream of consciousness interactions. One
neat thing I've noticed is Ruben's texts lend well to division by 80 Characters.
This implies that he frequently writes with words that are a factor of 80. Neat!
)

When the github repository goes live you will be able to find the raw files used
to assemble the corpus under Project_1/Raw-Data. They are labeled text files
according to their contents.

Now that we have all the needed plain text, it's time to assemble them.
Corpus-1.txt contains the Communist Manifesto and Dick Hardboiled. Corpus-2
contains the Communist Manifesto, Dick Hardboiled, and A Cyber Punk.

Now that we have the assembled data, we just need to follow the directions
in the [aforementioned article][6] to generate our network. The reason we are
using that notebook is because it gives us access to free compute power that
will dramatically speed up the learning process. It isn't as cool as doing it
by hand, but time is getting away from me, and having access to google's servers
for free is pretty chill.

I ran corpus 1 over 10 epochs. Took approximately 15 minutes to fully execute.

The dataset is so small that the system needs longer to learn. The following
output shows the process with only 10 epochs of iteration.

```
Training new model w/ 4-layer, 128-cell Bidirectional LSTMs
Training on 91,072 character sequences.
Epoch 1/10
88/88 [==============================] - 34s 389ms/step - loss: 3.6865
Epoch 2/10
88/88 [==============================] - 31s 355ms/step - loss: 2.5044
####################
Temperature: 0.2
####################
e the of the the at ont the the the the the the the the that the the the the the ant the the the the the the the the an the the tit the the ant the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the allint the t

he ant the atist the ther the tit at the the the the the the the the that the the the the the the ond the the the the an the the the the the the the the the the the inge the thes the the the the the the the the the and the the the the the the ant the the the the the the the the ant ant the the the t

he the the to the the the the the at the the the at the the the the the the that the the an the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the at the the the alle the the the the the

####################
Temperature: 0.5
####################
r tande the praly ont to tha tre anses atse taint at the the at ins the ang tit the alet tinge therre theat the at the of
ride ter pere ind tians thetterat pe iret there annt ant an tilouritis the ond thee theast tit the ante hal thatt tite to the the thationt pret, atitinge iteve dorectit thes
cess

thers ato tourer the toant the antiald the ang the tas the ang inge, catralationt ses all ont the the letet to the tuerte the tater itare ant at thiond I an stes thed indetins the thes the thed the and al thes acaren theres ind the te then thitast the the a anta the re on the
cke tar of the ter al t

end to pe ingentis the theins ther pent thestitint as issit alllind and thexat the the thet the the ant ant at thete erorin ta ofr tallse ithe the tint ant atiallind thet ants, allen the the to of the theeis thin thess thary the is the ancet an pay te the then mis fate olind tourind thel the thed as

####################
Temperature: 1.0
####################
a.

betess dete an ta. I of loto te Saved,, The iny ono tiousge
Acet mt moweve
I lI is tiherlpet BeLtay, onctly batitysmpemdt kut, da Fre thalt ta buls larisstam staliase rins sothestin the cheith itimet,
 do featialyin ancer the eptl plaoclouty fo aiteieboc she. Siis
ank.

I trours tthe aboumawatl

nd toe ive of
ctm
“Wes, oftian atamel, iy PI mut die, ar sad.
It this couleicol wie tarinten
bomabulawa tra piitio pan tareroftisttt I tret sithienk
Ivase
ant
vurs I IxEcjushat as. mole
tid atane las se thancitind, tho caporinnskin worre.
plabed grery ther iomat at oveang an.

ketres ave I Thied ap

Hen mtory anticogeied, ad, ediss tecitho ithereby tis
buto
Belkenst alte ties aiy ste ase timy, mtatehemingighe, mes, there way,
ither thiss.

coont cand ito e, twy groas.

eda iewpter,
antal ic tf t.
 lied med, talariont,
rs.
 mtat sas tadse
The Yarre cte semptoropahat ialptotse, to spaprans… wacay

Epoch 3/10
88/88 [==============================] - 31s 355ms/step - loss: 2.3817
Epoch 4/10
88/88 [==============================] - 31s 353ms/step - loss: 2.2756
####################
Temperature: 0.2
####################
he the the the the the the the the the and an and an and an and the and and an the the an the the and and and and an the the the the an and the the the the the the the the the and an the the the an the the the and an the an an the the the the the the the the the the the the the the the the the the a

e the the the an and and the an the the the the the the the the of the soce and an the the the the an the and the the the and and an the the the the the the the the the the the the the the and the the an the the the the the the and an the the the the the the the the the the the al the the and and an

of the and the the the the the the and and and an and al and the the an the the of an an the an of the and and the an the the an the the the the the the the the the al an the the an an the the the the the the and an and the and an the and an an the the the the the the the the the the the the the the

####################
Temperature: 0.5
####################
he in the and than polly and at on and an ang of the of an the thing manercall an pan an cond ad laris ald the of we the the the of the the con in the an the the the the an the the the and to an the socend the beve the the and in and the an the the the ape se the of ach and the the the the the is ay

on fin cans and and as sitry the an as of an of to the the trall the an and the cond at the and thit the, the the the far of as to abore an sed the as to popectan at abere to the an the the the so pen the to and ind of al of socing and an the ferty and all e of the the the the in an the antent ond f

ith the the an the in and in an the hang ant an to ande cand sontion acand in, the of al the ond the fin the ind the che an the the nes the the pre in and and and and all to the an alivermas ant efran aban of the the
ans the land al the in ton of of the of and bolly cant and to and an the coris inte

####################
Temperature: 1.0
####################
at a ince, aspon of cllitlal
I sen, the shatarsiasis, halin, itth berer, ducles nom.”
The fmads yo mewarily kinty an ond of peraterif henst worevisoss aldakel ior acs, ith kurgel opronge, on ag
the eprnif to an fatke poud the farissir collith-reain: sretacats ow she th Gerle
be any thanis alf crangh

rer. I rusap, aby a and banod foverrey wich ind wont me boled waby a inditicte bolf by giations ghaprpiticaser siltian as the the ad a tais of land on reut athes and gugne a them inucthendonng love yats, ton th is forked.

I ta the wie the ebome sorg of lory ftoicim and ectage.  Onuiss the ton er vu

cas ond batie, an so os and liciablery
ccicit ad ave th at’, and uved inkent smpeceve ughe whe fis don tat er thate ant sud efforestisingion, axpest lans fordetals
anme.
Th’n fin, tot diss ofmbald the an e and dis becoreryials a lagiat, them
cacey the ROiof ous a on feby, pnath Nin th goen exsht of

Epoch 5/10
88/88 [==============================] - 31s 355ms/step - loss: 2.1740
Epoch 6/10
88/88 [==============================] - 31s 353ms/step - loss: 2.0598
####################
Temperature: 0.2
####################
ll of the propertions of the bourgeoising of the proletered of the bourgeoisingeoising and are and in the bourgeoisingeoising of the prolered and and and of the prolered of the condionged of the prolertare and and and and and of the proletaricall and and of the sociation it of the bourgeoising of th

all and and the bourgeoising of the socially of the prolered and of the proletaricall and of the bourgeoising and and and and of the bourgeoising of the ware and social and and and the proletarial of the and and and the proletaries of the the conding and and the berection, of the proletaricall of th

e bourgeoising of the proletaricall the proletered of the bourgeoisingeoising of the bourgeoising and of and of the condere at of the bere of the contered and of the prolere and the sociall of the proletaricall of and and of the conderers and of the the prolentaring the conding of the proletaricall

####################
Temperature: 0.5
####################
iatingsicall
the nenther of the poluctioningeoiss.

The pracentry of the
goling the leargeois of the countion the momeral all
condernote of the nasertensens and and are as the socation.

The and age itsence of and courgeoisically of
the condernolition the of
mourgeoisingeoises of the
recainers the t

.  The prole, the condif lether the and the prerentorer the proletenticall the co manderaction
con and contered the socout of that beresetion in the and to ginto the bourgeoisinalially for at and in that the seace at bourgeoisition to and bere proletaly
con ragen pourgeoisition in the clation
gelath

d an sepistorial the ine on of the the ine in to mourgeoising cand the contung apreroped and mourgeoising the the andernere arestions abourgeois of meacentungeoision, a sule of bertorgeoisie. I prole the more safling the caring of approlopre at of and compilartion.

The bourgeoinition
the and with a

####################
Temperature: 1.0
####################
er conder of
graigion
sindo iar for ougllly. Thithe listlizatall.  Thoudsh DAllllg
nanke courne ce retill atser. It I in forgatings to
than furnerich-poivaingel, vasist ackeander.  Freack abardeing igneave to
acher mase of
a
moverc of
thee orelve, fin marecoret. Ind eay’t I’maef
a efrae the a yolved

onating wintsh pures the prolever themed pre an owst engaved curadecten. I fore ay letachiated of ital the free loithar.

“rominat, da deck!”
Incosetion" of by reegent gera weremominioust reeader.
 The parke of clacaderered the bugtgheiren wapionual hquin rithe mopestoracully cotiond dould’s ho clar

thole meantoretion. ’vend wheses hemomerlize.
E’s blolll and offichetery.

The madterainduing aganiby horefaricatal the niabusical are andernevend, dave reomenc
to
pentary ith rive, rig andemens an jus worm vare us phan’se alition.
 Therys then they ore bourgeoinsimales of iner casterepreteoas..”, e

Epoch 7/10
88/88 [==============================] - 31s 354ms/step - loss: 1.9494
Epoch 8/10
88/88 [==============================] - 31s 353ms/step - loss: 1.8507
####################
Temperature: 0.2
####################
n the working and the proletarial and in the proletaries of the proletaries of the proletaries of the conding of the proletaries and mover the proletaries the sellice of the proletaries of the proletaries and and the proletaries and in the modern of the proletaries of the wat and and and the and in

the proletaries of the conding of the proletaries of the bourgeoisie of the for in the proletaries of the modern of the proletaries of the modern of the proletaries of the competion of the proletaries of the proletariat of the proletaries of the proletariat and and the modentry and and the proletari

e modern of the proletaries society and the with and and and in the proletaries of the conding of the bourgeois in the proletaries of the proletaries of the proletaries of the conding of the proletariat and the proletaries of the proletaries of the proletaries of the proletariat of the modern of the

####################
Temperature: 0.5
####################
 the clolution and in the condings of the down the now proletaries introductions she proleved in social and of the and in the ore mover the bourgeoisies, the property and rastion of the condine worne.

And the mounder the she proletariat sitered the society, in the nevernoption the and and in a hise

ry histred cits of mench ine conderne the do class with the larling of the proletaring and the proletaring in the modern on the wat of my the wast of her the clansss of the leverty on the mere the in bourgeois and the ine productions of the
movent of the destrence nother the class, when and the into

ore in the pastion the country the prolutions of the reepentions of the whis clas for cleass of the proletarion of the conding in the class class achered society society it apprers, and lition mence of the proletariate, society factions the smenetion and by
modern the commention of the production pr

####################
Temperature: 1.0
####################
t sepres tingh proda,
They the isncited – I lollicazal seations and leath herrover of hizder hathed’tion, wat selyre ourde of thee sunn’in retently steansy.

The coundend and and and ove cind-alod.
 The
proletaringé
fat adident in labe tot and for pate. The agelopede the clallf the hallibal the clod

 tacel, an whis Fen the moabile her, the gected-veren an freforen the lacinticalilly. In jentalizally feat the well.  Thion you apond her prises of the smostiver of it iny
bearthing awituall purolever the stanbleciatly bentor feous, upe the naze out in antivent. Ho dion. Mand sins.” I lew nistay her

thee anal bourgeoismings the at pelode proder an
but
shear
be reash indietion the wactes, im?
 I heand whis Stas ust. I when’sel my treea sepre of the Socuce esindy.  In the haver forsered sou’ly and of an
the pornations withs cinsh wornountive no. Hhe
Ger the senc the ame of sother form and
dithh w

Epoch 9/10
88/88 [==============================] - 31s 354ms/step - loss: 1.7696
Epoch 10/10
88/88 [==============================] - 31s 352ms/step - loss: 1.7040
####################
Temperature: 0.2
####################
iat of the working of the proletariat of the counding of the proletariat of the bourgeoisie of the social of the proletariat of the proletariat the proletariat of the modern of the proletariat of the compintence of the proletariat of the comment of the proletariat of the proletariat of the proletari

e proletariat in the conditions of the proletariat of the proletariat of the proletariat of the working of the proletariat of the working of the proletariat of the proletariat of the proletariat, the proletariat of the modern of the modern of the proletariat of the proletariat the proletariat of the

rking the proletariat of the proletariat of the proletariat of the selfice of the proletariat of the countriations of the proletariat the proletariat of the working on the bourgeoisie of the proletariat of the social of the proletariat in society of the proletariat of the bourgeois the proletariat o

####################
Temperature: 0.5
####################
ordaw.

I class of the proletariat and it the classs of the proletariat has the cas. The itsurs of mans and becases, in the popcetent of the proletariat up class bourgeois condition the socialisty, in the to the workers of inderation the proletariat and of the modern and of the complete of the repan

he socialism of the bourgeoisie, the proletariat a society.

The country, and barling the carst the the working and the rever of ont the bourgeoisie of the prentence of the lother production, the bourgeoisie of the comment condition of the ore have sead inderation of and no her for its with mange th

n onterant of the modern the
French of all and in the proletariat of the proletarian of the the proletariat of the politial in the shanke shourgeois head of wall with inding of
standing of the complosition to was the down you existion of the canists of production of inting condernation the groletari

####################
Temperature: 1.0
####################
ongual moust, mut the Cohm more whom belopred that, the bourgeois ine of impepencant wivicums and
and
more to bourgeed us of the not like in the coin cramborer, but fir and polterians, with the useving one the suringain frieas, ce on the revolking pullivale predevenct in the worting a cocorets, no g

dms it the sonsing widss, the the fromen hich peconce of the onalilutions
crose dive and feudtory poterte,
fell bess mave stomence in heroms,
class and
miss of
untam, indeed had the clich but is drestroopres her, and
gontername musep-codustrie, the the wag.  They Oust when for by lasucts a socust "P

f mand to shaire sup the chifaped of but bemaning and of voing.

Neo lonje circtious yrair?
Gerss a’lly theypp foure on make fifferent slucks its, of abrongior its inuce,
der.

I a Neir
dis shey lorks you polition, raking where noger of her I moter squelling sacke?
The beemed
of the prolutical conpr
```

The higher the temperature the more creative the model is allowed to be. Around
this time I began to panic that there might not be enough data to properly
train such an advanced network.

I quickly decided to make some changes to the model. To begin with, I had been
training the model to learn how to spell. With such a small dataset, it would
likely be a better idea to teach it how to construct sentences. To do so, I set
the neural network to learn at the "word level" versus the character level.

To account for this learning change, I also modified the maximum number of
previous words the model would be allowed to consider before generating a new
word from 40 to 8. This is approximately the number of words in a basic
sentence.

Second, looking at our data reveals that it follows three very different
schemas, or formats. I had initially tried to train the model so that it would
learn forwards and backwards, but without more concrete and stable formatting
this was inefficient. I disabled bidirectional learning for the LSTM.

Finally, in QUINTUPLED the number of epochs to 50. I then executed the changes
against corpus 1. While this also produced highly amusing output, I spent
some additional time tweaking the neural network.

__Some time later...__

After playing with a bunch of settings, I ended up modifying a lot. The first
thing I did was increase the Hardboiled:Manifesto ratio in Corpus 1. Once
I increased the ratio, I duplicated the resulting text to get over 10,000 lines.

I then did the same thing and duplicated all the content in Corpus 2 until I
also had over 10,000 lines. Finally, I did a bunch of customization to the
neural network. At one point I had gone so focused that it was only capable of
learning the word of. Here's the resulting configuration:

```
model_cfg = {
    'rnn_size': 128,
    'rnn_layers': 4,
    'rnn_bidirectional': True,
    'max_length': 10,
    'max_words': 10000,
    'dim_embeddings': 100,
    'word_level': True,
}

train_cfg = {
    'line_delimited': False,
    'num_epochs': 700,
    'gen_epochs': 50,
    'batch_size': 1024,
    'train_size': 0.75,
    'dropout': 0.0,
    'max_gen_length': 300,
    'validation': True,
    'is_csv': False
}
```

The model is currently trianing, and I expect it to take approximately 3-4
hours. I plan to update this post with new information as I compile it, but
I do work tomorrow so I have to be responsible. :(


[1]:  https://twitter.com/urbanfriendden "Ruben's Twitter Timeline"
[2]:  https://twitter.com/urbanfriendden/status/1054101745134194688 "Birthday Tweet!"
[3]:  https://www.thefriendden.net/2016/10/20/the-adventures-of-dick-hardboiled-in-neo-noir-dark-noir-city/ "Dick Hardboiled Series"
[4]:  https://www.gutenberg.org/cache/epub/61/pg61.txt "Project Gutenberg's Copy of the Communist Manifesto"
[5]:  https://en.wikipedia.org/wiki/TensorFlow "TensorFlow Wikipedia Page"
[6]:  https://minimaxir.com/2018/05/text-neural-networks/ "How to Quickly Train a Text-Generating Neural Network for Free"
[7]:  https://atom.io "Atom.io Home"
[8]:  https://www.thefriendden.net/2015/11/07/dick-hardboiled-and-the-meaning-of-love/ "Dick Hardboiled and the Meaning of Love"
[9]:  https://www.thefriendden.net/2015/12/19/dick-hardboiled-has-an-off-day/ "Dick Hardboiled has an off day"
[10]: https://github.com/kennethreitz/twitter-scraper "Twitter Scraper"

# On Mechanism Design

> Some musings on mechanism design intended for all audiences.

I struggle a lot with explaining what it is I do exactly. I can talk about my job or certain projects I'm working on, but it's difficult to put it in the context of the industry I work in.

**I do mechanism design research.** I've gotten better at explaining what that can look like, giving examples of auction design or work that I've done with clients. However, I still can't get to what it is at its core, and why this is what I chose to do.

When I would start going into what a mechanism is, I usually started waffling on about how a mechanism is a "sort of system" between many "players" designed to benefit all these players in some way and that (hopefully) can't be gamed. I would quickly move on to examples before I lost whoever I was speaking to for good.

Proper mechanism design is critical in marketplaces. Taylor Swift fans and sneakerheads might not be familiar with mechanism design but if you talk to them about it they understand immediately why mechanism design is crucial.

I like [Vitalik's definition](https://nakamoto.com/credible-neutrality/) that mechanisms are algorithms plus incentives. It gives an idea of what a mechanism looks like in its most distilled form: it's an algorithm, a procedure of sorts. Most people are familiar with the word "algorithm" by now, and to the initiated, it provides a hint of the level of rigour that mechanisms are analyzed with. Adding incentives on top gives us a glimpse into what's at stake in these systems and the motivations of their participants. I want to hone in on this definition because Vitalik gives some good examples of mechanisms so you get the general idea and potential scale that mechanism designers work with, but he doesn't explain how mechanism design is done.

Why would a mechanism be an algorithm, and what kind of algorithm is it? What kind of inputs do these algorithms take? What does "plus incentives" entail and how can we design for that?

Let's start by going over the different players (also called agents).

There's the _Mechanism Designer_ (MD for short), this is who designs the mechanism and wants other players to use their mechanism (imagine we're talking about a marketplace here) for some reason. They're not normally modelled as an agent, but it would be a mistake to not account for them as they may be able to participate in their mechanism for their gain and the detriment of others.

There are honest agents, who participate as the MD intends players to. They are the ideal participants for a mechanism designer. If every agent were honest, mechanism design probably wouldn't be a field of study in the first place. However, they're worth including in our model as they are our most desirable participants and we want to protect their interests as the MD. Using a limited edition sneaker sale as an example, an honest player would line/sign up to buy a pair for themselves.

Next, we have rational agents, these aren't necessarily bad players but they are expected to do anything to advance their interests. This can mean using bots to quickly buy up the supply of some items (tickets, sneakers, etc...) and reselling it at a profit. For rational agents, other players’ needs aren't a concern as long as they fulfill theirs. This is what motivates the study of mechanism design. A mechanism designer wants to either restrict or properly incentivize rational agents into acting honestly so as not to ruin the fun for everyone else. When done right, rational agents have no better alternative than playing by the rules.

Lastly, we have adversarial agents. These are our bad guys who want to ruin the party by any means necessary. While not every mechanism will have to deal with this type of player, as someone would need the motivation to disrupt your system, it's important to know this type of agent exists and may try to throw a wrench in your mechanism. This is more of a security issue than one of incentive design, which MDs must keep in consideration nonetheless. DoS attacks are an example of what adversaries might try, which is why [gas](https://ethereum.org/en/developers/docs/gas/) is a thing on Ethereum.

A Mechanism Designer wants to host/offer some sort of marketplace or service. As a player themselves, they of course have their own _objective_ like making a profit, but they usually do care about the best interests of the players who use their mechanism. We call this social welfare and it is something we aim to maximize as mechanism designers since we also want people to use our mechanism. No one will use our marketplace if they don't get anything good out of it too, and we won't make any revenue as a consequence.

Examples of objectives a MD could have are:

- maximizing revenue from the sale of an item and ensuring the items sold go to honest agents ([auction theory](https://en.wikipedia.org/wiki/Auction_theory))
- decreasing congestion on roads and networks ([congestion games](https://en.wikipedia.org/wiki/Congestion_game)) they want people to use
- ensure fairness and incentivize agents to behave honestly

Honest and rational agents want to use our marketplace with their objectives in mind. Recall that the MD can be one of the honest or rational agents. Adversaries are easy enough to model, their objective is to increase others' loss. Each player also has a set of possible actions, we can refer to this as a strategy space. Modelling objectives is usually the easiest part, but modelling the strategy space is where MDs usually get tripped up and don't account for possible errors in their design that can be exploited. Not all players are honest, and some may exploit any vulnerabilities in your design to further their goals.

You can't just sell a limited run of valuable sneakers and expect regular people with a normal wifi connection or 9-5 to be able to buy these easily. Rational resellers will develop state-of-the-art bots or pay teens to stand in lines if it means they can resell them for a greater profit. Companies like Nike must take care to design a sale that gives everyone a chance to cop some sneakers at a fair price while not being too susceptible to spam by bots. [Raffles](https://queue-it.com/blog/sneaker-raffles/) and solutions like [EQL](<[https://www.eql.com](https://www.eql.com/)>) are a step towards [fair sneaker drops](./sneakers.html).

A mechanism designer wants to produce some outcome(s) that aligns with their objective and benefits its users as much as possible. Some players might gain at the expense of others but as long as most people are happy, our job as MD is done. Going with the marketplace example again, the outcome produced will be the final allocation of goods after the exchange occurs. The MD will want to let users exchange goods with each other in a way that maximizes the marketplace's revenue while also upholding some level of fairness between every player (not privileging bigger firms for example).

The mechanism designer must take all this into account and design a procedure to _systematically_ produce an ideal outcome for themselves and the other agents. This necessarily implies designing and expressing the mechanism as an algorithm, since we can leverage the many tools and techniques computer scientists have developed for formally proving desirable properties of algorithms. Notably, a mechanism designer will want to produce a provably optimal outcome.

Once the MD formulates their objective precisely enough (say as a mathematical function), they can prove whether the outcome output by the algorithm maximizes the objective function (making it optimal). Computer scientists call this property _correctness_. We can also analyze algorithms in terms of efficiency by finding out how many steps it takes to compute the outcome, how much players have to communicate/interact amongst themselves and with the MD to use the mechanism, and how much data agents must hold while using the mechanism. These measures of efficiency are known as time, communication and space complexity respectively, and they are a central focus of study in computer science.

The inputs in this algorithm are different from the static inputs seen in intro comp sci courses. In our case, the inputs are the _players_ themselves along with their strategy spaces. Given the current state of the mechanism, each player will choose their best course of action and the mechanism must produce the next outcome according to some predefined procedure. Naturally, each player can either choose to be honest (which is what we want as MD), or they can _deviate_ and do whatever is in their best interest instead. If no player is incentivized to do anything but be honest then our mechanism is well designed and we can call it a day. Proving that is the hard part. :)

That being said, how do we also convince our users, the people we are trying to attract to our platform, that our mechanism does what it set out to do?

Normally, an outsider, not privy to the precise algorithm the mechanism uses, would have to infer how the mechanism is designed and what its objective is by observing the outcomes it produces. This is the good ol' [**POSIWID**](https://en.wikipedia.org/wiki/The_purpose_of_a_system_is_what_it_does) heuristic. It's a good heuristic for calling out an obviously bad system (it does not work as stated), but it can't convince anyone that a system incentivizes people to act honestly.

Moreover, ordinary users can't even tell if the _mechanism designer_ is playing fair since the source code of the algorithm and actions of other users are hidden from all but the MD. Recall that the MD has their objectives and could act unfairly to further their interests (at their users' expense of course). For example, Google has gotten slammed with an [antitrust suit](https://www.justice.gov/opa/pr/justice-department-sues-google-monopolizing-digital-advertising-technologies) for manipulating the Google Ads auction to increase their profits.

A natural next step in the field of mechanism design would be designing a mechanism where even the creator of the mechanism cannot influence outcomes in a way that benefits them. This property is called _credibility_. This is hard to do since the algorithm is typically closed-source and hosted by the mechanism designer so it has a privileged view of all incoming actions as well as the ability to _censor_ actions or at least delay them. Imperceptible censorship and delays could be used by a mechanism designer to manipulate an auction, or to trade against users on an exchange.

This sets the scene for what blockchains can do for mechanism designers and users.

Blockchains (like Ethereum) allow for the creation of _truly credible_ mechanisms by providing an environment where programs can be hosted outside of the control of a single intermediary. Once deployed, the source code of the mechanism can be open-sourced and verified, and it also can’t be taken offline. Blockchains can offer these guarantees thanks to properties called: liveness, censorship resistance, and source code verification. From there, fairness, [strategyproofness](https://en.wikipedia.org/wiki/Strategyproofness) and [incentive compatibility](https://en.wikipedia.org/wiki/Incentive_compatibility) can be proven directly and rigorously. Chitra et al. have a great paper on how blockchains can enable [credible and optimal auctions](https://eprint.iacr.org/2023/114.pdf).

In the antitrust example I gave above, the DOJ was able to step in and put a stop to Google's dishonest actions. It isn't always feasible for the law to step in, especially across borders. On a blockchain, new mechanisms can be deployed and serve credibly, and on a global scale on Day 1.

It's a simple idea, but also incredibly powerful with huge political ramifications. Bitcoin, a completely permissionless and decentralized digital currency, was the first application of blockchain tech. It showed that this idea was powerful enough to power a whole new form of money.

The part that excites me the most about the blockchain space is the fervent energy and belief that we can shape and build our own systems and infrastructure of tomorrow, and it can _provably work for us_. I believe so too, and I want a hand in shaping that future.

---

I want to thank Azizah Hossain and Ninaad Kalla for their invaluable feedback on early drafts of this post.

One of the comments Azizah left was interesting and worth highlighting:

> Isn't the purpose of creating a mechanism is to profit one way or another? Are companies still gonna create mechanisms if they have little to no control over them? I'm not sure why this is naturally the next step. I kind of get why it should be but would this really come to fruition?

Crypto has been showing how profit and control don't have to go hand in hand. I haven't questioned why a business would do this since I'm in too deep at this point, but it's definitely non-obvious for anyone not in the crypto space. Unfortunately, it's out of scope for this article and I couldn't find a natural way to broach the topic, so I'll just defer to Jesse Walden's excellent article on [Crypto Business Models](https://a16zcrypto.com/posts/article/crypto-business-model/) for those who are curious.

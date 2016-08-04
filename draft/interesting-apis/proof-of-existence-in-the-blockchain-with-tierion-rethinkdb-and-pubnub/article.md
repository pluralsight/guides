In this tutorial we're going to build a web application very similar to [Proof of Existence](https://proofofexistence.com) and [Origin Stamp](http://www.originstamp.org/) using [Node.js](https://nodejs.org/en/), [Tierion](https://tierion.com), [RethinkDB](http://rethinkdb.com/), [PubNub](https://www.pubnub.com), and [jsreport-core](http://jsreport.net/learn/nodejs).

In a few words, these sites allow you to store the hash of a file in Bitcoin's Blockchain so anyone can certify that the file existed at a specific time.

This has the following advantages:
- It demonstrates data ownership without revealing actual data.
- It checks for document integrity.
- Using the blockchain, you can certify the existence of the document without the need of a central authority. 
- In practice, this decentralized proof can't be erased or modified by anyone.
- It's anonymous, no one knows who you are or your data.

This [page has a better explanation about what is Proof of Existence](http://www.newsbtc.com/proof-of-existence/) and some sites dedicated to it.

Our application will have the following functionality:

You can upload a file by choosing it or dragging it to the marked area. Then, its SHA256 digest will be calculated so you can send it to the Blockchain using [Tierion Hash API](https://tierion.com/docs/hashapi):

![Create hash](https://raw.githubusercontent.com/pluralsight/guides/master/images/c785866d-69cf-4cce-8c93-675d907de3f8.gif)


Notice how the list of last documents registered is updated real-time, we'll be using [PubNub's Storage and Playback](https://www.pubnub.com/products/storage-and-playback/) functionality to implement that.

The hash of the file and the Blockchain receipt given by Tierion is stored in RethinkDB, a NoSQL database with real-time capabilities. When Tierion alerts us that a block of hashes has been processed, using [RethinkDB's changefeed](http://rethinkdb.com/docs/changefeeds/javascript/), we update the verification page (and the list of verified documents) in real-time:

![Confirming Hash](https://raw.githubusercontent.com/pluralsight/guides/master/images/6963ff66-7806-417c-9b58-063c23946054.gif)


Once the hash is anchored to the Blockchain, we can check it using sites like [Block Explorer](https://blockexplorer.com) or [Blockchain.info](https://blockchain.info) and get the receipt in PDF format using jsreport-core.

You can also verify that a file has been certified in the Blockchain by selecting the *Verify* option:

![Verify file](https://raw.githubusercontent.com/pluralsight/guides/master/images/b061a641-34f1-4af6-a0a2-bce474421557.gif)


The entire source code to follow along with this tutorial is on [Github](https://github.com).

However, before diving into the code, let's explain what is the Bitcoin's Blockchain (or just the Blockchain).

# The Blockchain
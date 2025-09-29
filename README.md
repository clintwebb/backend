# backend
A backend service that can present anything without any custom functionality required.
Data is stored securely in the back-end, but processing of that data is all done at the front-end.

## General Purpose
The general purpose is to provide information or tools for people to access and use, along with storing some information which is really only related to them.

### An example
Someone wants to create a website which allows people to do some calculations.   Lets imagine it is something to do with house repairs.
They present a static website which has all the html/css/javascript etc in it.
When people want to use the website, and they are putting in information on it, the browser side talks to the backend and creates an encrypted chunk of data to keep the information.
The key for encryption is stored in the users browser.  Additionally, could also set it so the user can create an account with a password, etc, and the key is stored on the backend (but requires login to access the key).
The main purpose, when that person closes the browser, and then later on goes back to this website, it pulls the information they previously stored from the backend and re-presents it back to them.

It can be done at a very small level, or at a very large level.

### Another example
This is the real example, of the main reason for wanting to do this backend.   Want to create a website to handle a mini budget.  Almost all the data stored only applies to the user, not needing to be stored in a complicated database or accessing a bunch of shared info.

#### And the real implmentation of such an example.
 
 1. The backend service is installed on a 'server', and it will be configured with the server DNS names, and potentially certificates and so on.
 1. The front-end website development will be created, and can be stored in a different git repository, or even just pointing to a file storage location.  Multiple options.
 1. The scripting on the front-end will include some modules or components which can talk to the backend.
 1. For the site, new users would create an account, so that all handled on the front-end scripting, but on the backend it also barely included.  New accounts will be encrypted and a chain created.. that account can then access that chain.  The chain is really just a chain of blocks of data.  The backend will not know at all what information is in the block of data.  It will just provide it when requested.
 1. The block of data is encrypted with a key that was created in the browser and stored in the browser.  But for safety of accessing it from multiple browsers, the key can be password protected and stored in the backed, but requires some way of verifying the requestor before providing it.  This would normally be like a username/password or some other more modern authentication.
 1. The backend just provides blocks that are requested.  The front-end can extract the data out of the block using the keys that it has.
 1. The front-end can also create more chains of data.  Normally for different kinds of information.
 1. So in this example, when a new user starts to use the mini-budget website, and create an account, on their browser it creates the private key, requests a block with an identifier that it can determine from the account name, or whatever the front-end chooses.  That block can then store data that references any other blocks.
 1. The data being stored (and encrypted before it even provided to the backend), can be in any format required by the front-end.  Typically (and supported) is JSON formatted info that can then be processed and accessed easily. The info in that data received is normally what is useful to then reference other data chunks needed.
 

# Data types.
 1. Zones - this will likely be renamed to something else, but will often be associated with isolating different services or products.  The idea is that data that is related to a specific thing will be completely isolated (at the backend).
 1. Chunks - this is a single chunk of data that is referenced by a key.
 1. Chain - This is a series of chunks of data that are basically a chain of data.  Similar to a set of records.  When some data is saved, it is added to the end of chain.  This is mostly for historical or recorded information. Chunks can be used individually to do a similar thing, but the main purpose of the chains, is that the backend will know the different chunks, and can provide multiple of them in a request.  Eg, the front-end says, send me the last 20 chunks in the chain all in one go (to reduce the bandwidth and delay)... or could ask for the first 20 chunks... etc.    If not done in the chain, the front end can have a single chunk, which then referenes the next chunk, which references the next and so on... but each chunk would then require an extra request, which can then result in delays and complication.  The only thing different in the backend side, is that each stored chunk also includes a reference to the one before and/or after it.   To make it simple, in most cases... a new chunk is added to the chain, and the backend then stores and caches the chain list.

 


## Layers

 1. Interface (Provide Static content, API for data, proxy forwarding)
 2. Cache
 3. Storage


## Infrastructure Scalability
The goal is for it to work fine with a single instance, or split into multiple layers and spread.

## Caching
Normally only needed when multiple backend storage servers exist, or seperation between back-end and storage.  When blocks are requested, and retrieved from storage (seperate server), it can cache a certain amount in memory, and some in local storage.  Very useful in that blocks are never modified, only the chain of blocks is.  The amount of memory and disk caching can be configured, and in some ways flexible.

### Caching Performance and Monitoring


## Other interactions
This is planned for sites where it can store isolated information for accounts... but in reality, a lot of websites will need to interact with other services.  Eg... a database, or other data... along with other API's.  The backend itself is not intended to be used to run code, but there will be cases where it behaves like a reverse proxy to direct traffic to other services.

## Read-only streams
For the website, a lot of information will be specific to the account, but often would need access to other information.  This other information can be presented in a chain, but can only be read, and not updated.  To accomplish this, the stream will contain data that can be decrypted by a key provided, but a seperate key is used to encode other parts of the data, which sets up the chain.  When the chain is updated, a part of the information (linking the chain) is encoded with the read-key and the write-key.  The write-key is what is needed to add new chunks.



## Co-ordination between backend interfaces and storage.
When requests are received, they are processed on the interface and it needs to retrieve the requested data.  If it is cached, will access it there, but if not cached, then it will need to talk to the storage asking for it.   When initialised, the backend will create a network connection to the other backends, and will queue requests through it.   Normally, the number of backends available, will be useful in determining which backend to send the request to.   If there are 3 backends, each block will be stored on one of them, and backed up on another.  When a query is made, it will know which is the primary by using a modulus.   Each block is referenced by a number (based on a hash), a modulus of that number based on the number of backends will indicate which one is the primary.



## When storage server counts drop.
If we had 3 storage services, and 1 is dropped.  A recovery needs to be in place where the data is re-synced.  It will be in a temporary state where it will continue to process as much as it can as normal, but will re-arrange the structure with the other nodes to get it all re-aligned.  Eg, if a server dies, and the backup one now needs to know it is the primary, and another node needs to obtain the backup copy... it all needs to re-sync.


## Real World Examples (of Scalability)

For example, could start with a single server which hosts the front-end (which presents the web-page external content), and the accounts and data are stored as secure blocks in the backend, and then later when more clients are using the service, they decide to spread it over two servers.  Depending on the requirements, could be set so that the data is spread over both servers and synchronised...  but frontend and backend both in the two servers.

Then they move it forward in the future, where they split it, so that there are two front-end servers, and 3 backend servers.   THe data blocks on the back-end servers are split so that for each block there is a primary on one back-end server, and a secondary on another back-end server, and it spread around.   For performance reasons, it could be configured so that certain requests which are commonly received, will be sent to a specific server which contains those blocks.

The backend will not know any content of the blocks, but will know which ones are the most often requested, and need to be cached.

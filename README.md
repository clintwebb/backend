# backend
A backend service that can present anything without any custom functionality required.
Data is stored securely in the back-end, but processing of that data is all done at the front-end.


## General Purpose

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
When requests are received, they are processed on the interface, and it needs to retrieve the requested data.  If it is cached, will access it there, but if not cached, then it will need to talk to the storage asking for it.   When initialised, the backend will create a network connection to the backend, and will queue requests through it.   Normally, the number of backends available, will be useful in determining which backend to send the request to.   If there are 3 backends, each block will be stored on one of them, and backed up on another.  When a query is made, it will know which is the primary by using a modulus.   Each block is referenced by a number (based on a hash), a modulus of that number based on the number of backends will indicate which one is the primary.



## When storage server counts drop.
If we had 3 storage services, and 1 is dropped.  A recovery needs to be in place where the data is re-synced.  It will be in a temporary state where it will continue to process as much as it can as normal, but will re-arrange the structure with the other nodes to get it all re-aligned.  Eg, if a server dies, and the backup one now needs to know it is the primary, and another node needs to obtain the backup copy... it all needs to re-sync.


## Real World Examples

For example, could start with a single server which hosts the front-end (which presents the web-page external content), and the accounts and data are stored as secure blocks in the backend, and then later when more clients are using the service, they decide to spread it over two servers.  Depending on the requirements, could be set so that the data is spread over both servers and synchronised...  but frontend and backend both in the two servers.

Then they move it forward in the future, where they split it, so that there are two front-end servers, and 3 backend servers.   THe data blocks on the back-end servers are split so that for each block there is a primary on one back-end server, and a secondary on another back-end server, and it spread around.   For performance reasons, it could be configured so that certain requests which are commonly received, will be sent to a specific server which contains those blocks.

The backend will not know any content of the blocks, but will know which ones are the most often requested, and need to be cached.

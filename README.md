# Zero-Surveillance Online Age Verification
A Beag Tech mini white paper

Many countries are finally coming to grips with how unhealthy social media is for young people. What we need most are social platforms that are healthy for everyone, including children. However, it is also true that children are especially vulnerable. The path of least resistance amongst governments seems to be to rely on big tech providers whose solutions involve collection of sensitive identity documents and biometrics. Other solutions involve input of highly sensitive information and sharing of images into opaque digital wallets, creating a new high-value security target on the device or via app-store hoaxes and phishing.

Here are three alternative ways proposed to keep children safer online: 
1. Grant government contracts to firms who then access sensitive information for all adults. This is madness but it seems to be the default plan due to lobbying power and dollars involved.
2. Mandate that devices have a child/teen/adult flag (especially phones), e.g. configured by the account owner per mobile phone on a phone plan. Although this idea is not fleshed out below, it's another example of alternatives that should be discussed before more surveillance contracts are issued!
3. Find ways to leverage *existing trusted relationships* in a simple workflow that results in a "yes/no" response to age verification (i.e. no new personal information is shared with anyone). This is discussed below. 

We note that:

1.	People already have trusted custodians of their identity and age information in the form of banks, health insurance firms, government agencies such as Revenue, mobile phone companies and utility providers or even county library systems. 
2.	These existing institutions have websites with personal login requirements. In the case of banks, phone companies, and Revenue especially, an adult would be unlikely to share their login with minor children. 
3.	Answering the question “is this person of age” is a very different question (answered by “yes or no”) from “what is this person’s government ID number”; no one should need the answer to the latter. 

We therefore propose the following user experience. 
1.	In the social media app, the person sees a prompt to “Click to verify age requirement using a privacy-preserving third-party website.” 
2.	At the third-party website, the user is given a choice of participating institutions plus a 7-letter verification code e.g. ghx-78rt. 
3.	The person logs in to their institution’s account, clicks the “verify age” banner, and inputs the code ghx-78rt. 
4.	Upon typing the code, a page is launched redirecting to the third-party website saying verification is complete and that it’ll redirect to the social site in 5 seconds. 
5.	Optionally, the page could offer to save the “of age = yes” credential securely in their web browser’s cache for use with other social sites. Or click Continue to return to the social media app. 

Minimizing who knows what in this flow:
1.	The third-party website receives only a single-use code from the social site to start the flow (as mentioned below). Then it only receives an “of age = yes” response from the institution. It never receives any PII. After 10 minutes, the server discards even this scant information that it received. 
2.	The institution receives only the code e.g. ghx-78rt (which the third-party website discards in a matter of minutes anyway). 
3.	The social media app receives only a digitally-signed attestation of “of_age=yes” (issued by the third-party and tied to the original single-use code generated for the social account). 

Revenue opportunity for institutions:

Perhaps the banks, phone companies, county libraries, Revenue, and other institutions might charge the social media company for this service, perhaps at government-sanctioned rates that reflect historical harms caused by the social media firm to children, or paid out of a pool of fines levied to social media firms who have caused harm.

Sample technical implementation (sample code to be provided soon):
1.	Social site detects country of the logged-in user as requiring age verification.
2.	Social site generates a single-use code (“social code”) and associates it with the user account. 
3.	The person clicks the Verify Age button.
4.	Social site’s button launches the third-party website via a 302 redirect, passing the social code, the social site name, and the request type (e.g. age 16+). 
5.	The person learns about the process. They learn that this site will receive no PII. Nonetheless, they will be asked to set an on-device password for the credentials that they are generating (browsers are often de facto multi-user).
   
    a.	This password also allows local storage of the ECDSA private keys, discussed next, using encryption in IndexedDB (i.e. locally, never leaving the browser), so they can be used when the user switches tabs.
  	
    b.	And the encryption via IndexedDB also allows support for “remember this credential” capabilities later.
  	
    c.	Note: The password step is not a strict requirement, but it does allow for a clean multi-tab experience (users rarely finish an authentication flow in a single tab, so the keys should ideally be stored, therefore encrypted).
  	
6.	Third-party site’s client-side code generates three ECDSA key-pairs in the browser (one key-pair per party involved, i.e. for sending signed communications to the social site, the institution, and its own server) via the built-in “subtle crypto” library of any web browser. The private keys will be for digital signing and the public keys for signature verification.  

7.	Third-party site’s client-side code transmits to its server: all 3 public keys plus the social code and social site name (signing the transmission via the key intended for use with its server). The server holds this temporarily, e.g. in memory.
   
    a.	Included in this transmission will be a document intended for use by the institution, comprised of: (a) the public key generated by the client for use with the institution plus (b) the request-type (e.g. age 16+). Note that this is signed by the client, i.e. using the public key intended for use with the institution.
  	
    b.	Third-party site’s server will sign this document and generate from its hash a 7-letter alphanumeric code (of which there are over 84-billion combinations) that can be used later by the institution to retrieve this document. (This later saves the user from having to rely on copy/paste or passing large query strings to the institution’s login page). 

8.	The person now follows instructions to: pick a participating institution, log in, click Verify, and enter the 7-letter code. 
9.	The institution’s server-side code authenticates itself via secure API credentials and uses the 7-letter code to fetch from the third-party site the document.

    a.	It verifies that the content of the file does in fact imply that 7-letter code (i.e. based on hex math on the hash of the content).
  	
    b.	It sees the public key was used to sign it. 

10.	The institution can now digitally sign a document indicating: “the owner of the private key corresponding to the public key seen in this document (hash-addressable via that 7-letter code) is in fact 16+”. 
11.	The institution transmits that as a signed document to the third-party server. 
12.	The third-party site’s server links that document with the rest of the request and replaces all other files with a final signed file to the effect of “the owner of the private key corresponding to the public key associated with the social site is 16+, and this certificate is valid for the (unknown) social account represented with the anonymous social code xyz”. 
13.	The person now sees their browser launch a new tab of the third-party site. 
14.	The third-party site’s client code resolves to the server-side document. The client digitally signs this with the key-pair intended for use with the social site. 
15.	The third-party page now redirects to the social web page, passing the signed token (e.g. JWT).
    
    a.	An encrypted cache of the credential is optionally cached in IndexedDB. (More specifically, this would need to be a ZKP credential issued by the server that links the public key with the age verification. That’s a subtly different document from what gets presented to the original social site.)
   	
    b.	For the next social site requiring this proof, the client can transmit the credential to “remind” the server of what it digitally signed and can therefore rely on. That prior issuance together with a signed request (using the same public key) to link the new social code to the required credential can fulfill a “no touch” process for the next social account.
   	
17.	The social site verifies signatures (the client’s and the site server’s) and issues the credential. 
18.	After 10 minutes, everything is deleted from the third-party’s server as part of a periodic job.

Flow Diagram:

PHASE 1: SETUP & KEY GENERATION (Steps 1-7)
```
--------------------------------------------------------------------------------
[SOCIAL SITE]          [USER BROWSER]           [WHOS.IE SERVER]       [INSTITUTION]
     |                        |                          |                    |
     | 1. Detect Need         |                          |                    |
     | 2. Gen "Social Code"   |                          |                    |
     |                        |                          |                    |
     +--(302 Redirect)------->|                          |                    |
     |  (Code+Site+Type)      |                          |                    |
     |                        | 5. Prompt for PIN        |                    |
     |                        |    (Encrypt IndexedDB)   |                    |
     |                        | 6. Gen 3 Key-Pairs       |                    |
     |                        |    (Social, Inst, Srv)   |                    |
     |                        |                          |                    |
     |                        +--(7. PubKeys + Code)---->|                    |
     |                        |    ***Signed (Key_Srv)***|                    |
     |                        |                          | 7a. Create Doc     |
     |                        |                          |    (PubKey_Inst)   |
     |                        |                          | 7b. Gen 7-Letter   |
     |                        |                          |    Code (Hash)     |
     |                        |                          |                    |
     |                        |<-(Code + Instructions)---+                    |
     |                        |                          |                    |
```
PHASE 2: THE BANK HANDSHAKE (Steps 8-11)
```
--------------------------------------------------------------------------------
     |                        |                          |                    |
     |                        | 8. User Opens Bank       |                    |
     |                        |    (New Tab/Window)      |                    |
     |                        |    - Logs in             |                    |
     |                        |    - Enters 7-Letter Code|                    |
     |                        +--------(Code)----------->|                    |
     |                        |                          | 9. Fetch Doc       |
     |                        |                          |    (API Auth)      |
     |                        |                          |    - Verify Hash   |
     |                        |                          |                    |
     |                        |                          | 10. Sign Attestation
     |                        |                          |    "Owner of Key_I |
     |                        |                          |     is 16+"        |
     |                        |                          |                    |
     |                        |                          +--(11. Signed Doc)->|
     |                        |                             (Server-to-Server)|
     |                        |                          |                    |
```
PHASE 3: COMPLETION & REUSE (Steps 12-17)
```
--------------------------------------------------------------------------------
     |                        |                          |                    |
     |                        | 12. Server Links & Swaps |                    |
     |                        |     Doc for Final Token  |                    |
     |                        |                          |                    |
     |                        | 13. User Returns         |                    |
     |                        |     (New Tab/Refresh)    |                    |
     |                        |     (Unlock via PIN)     |                    |
     |                        | 14. Sign with Key_Social |                    |
     |                        |                          |                    |
     |                        +--(15. Final JWT)-------->| (Optional: Cache)  |
     |                        |     (Redirect)           |                    |
     |<-(15. Pass JWT)--------+                          |                    |
     |    (Query String)      |                          |                    |
     | 16. Verify Signatures  |                          |                    |
     |     (Client + Server)  |                          |                    |
     |     GRANT ACCESS       |                          |                    |
     |                        |                          |                    |
     |                        |                          | 17. DELETE ALL     |
     |                        |                          |     (10 Mins)      |
     |                        |                          |                    |
```


# Find the leaking bucket!

## Description
It was made known to us that agents of COViD are exfiltrating data to a hidden S3 bucket in AWS! We do not know the bucket name! One tip from our experienced officers is that bucket naming often uses common words related to the company’s business.

Do what you can! Find that hidden S3 bucket (in the format “**word1-word2-s4fet3ch**”) and find out what was exfiltrated!.

[Company Website](https://d1ynvzedp0o7ys.cloudfront.net)

## Hint
Encrypted files are not always safe

## Solution

The website looks pretty empty except for a few words, and a quote by Steve Jobs. We know that the s3 bucket has a url in the following format: 
```http://word1-word2-s4fet3ch.s3.amazonaws.com```


With the hint given in the challenge description, we first create a wordlist by copying all the words in the website and placing this in [wordlist.txt](wordlist.txt). We can then use [wfuzz](https://wfuzz.readthedocs.io/en/latest/) to fuzz for the bucket:
```bash
$ wfuzz -w wordlist.txt -w wordlist.txt --hc 404 http://FUZZ-FUZ2Z-s4fet3ch.s3.amazonaws.com
```
We have used the flag ```--hc 404``` to hide all requests with an error 404. We also specify the wordlist twice since we have 2 parameters to vary.We eventually get the following response:

```bash
********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://FUZZ-FUZ2Z-s4fet3ch.s3.amazonaws.com/
Total requests: 2304

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                
===================================================================

000001700:   200        1 L      5 W      465 Ch      "think - innovation"                                                                   

Total time: 120.9198
Processed Requests: 2304
Filtered Requests: 2303
Requests/sec.: 19.05393
```
This tells us that it is likely to be 
```http://think-innovation-s4fet3ch.s3.amazonaws.com```
since it does not return a error 404. Navigating to the website gives us the following XML:
```XML
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<Name>think-innovation-s4fet3ch</Name>
<Prefix/>
<Marker/>
<MaxKeys>1000</MaxKeys>
<IsTruncated>false</IsTruncated>
<Contents>
<Key>secret-files.zip</Key>
<LastModified>2020-11-17T15:59:54.000Z</LastModified>
<ETag>"ac4f39a2bb4c6a4e495bb8819ff8fd39"</ETag>
<Size>273804</Size>
<StorageClass>STANDARD</StorageClass>
</Contents>
</ListBucketResult>
```
We can download the file by navigating to 
```http://think-innovation-s4fet3ch.s3.amazonaws.com/secret-files.zip```. We can use 
```bash
zipinfo secret-files.zip
```
to obtain 
```bash
Archive:  secret-files.zip
Zip file size: 273804 bytes, number of entries: 2
-rw-r--r--  3.0 unx       50 TX stor 20-Nov-17 22:59 flag.txt
-rw-r--r--  3.0 unx   275299 BX defN 20-Nov-17 18:15 STACK the Flags Consent and Indemnity Form.docx
2 files, 275349 bytes uncompressed, 273360 bytes compressed:  0.7%
```
Unfortunately, the file has a password. We tried to use [fcrackzip](https://github.com/hyc/fcrackzip) to get the password using our original wordlist, as well as modified wordlists such as "word1-word2" or "word1-word2-s4fet3ch". This did not work, so we tried to see if there was any passwords hidden in the photos using steganography but we could not find anything as well.

At this point, it was nearing the last 3 hours of the competition and we had been trying to crack this for 1.5 days so we decided to buy the hint. The hint told us that it would be possible to break this even without knowing the password. We searched around and found out about the [Biham and Kocher's known plaintext attack](https://link.springer.com/chapter/10.1007/3-540-60590-8_12). This made use of the fact that we know some of the plaintext in the file.

From here we can proceed in two different ways:
## Method 1: Flag Format
We know that the flag format is in the form **govtech-csg{** so we can create a text file called ```flagf.txt``` with content **govtech-csg{**. We do not have to ```zip``` this file since we have deduced earlier that the ```flag.txt``` file is not compressed, so the bytes used for the plaintext attack are the same. We can then use [bkcrack](https://github.com/kimci86/bkcrack), a tool written to crack encrypted zip files using Biham and Kocher's known plaintext attack.
```bash 
$ bkcrack -C secret-files.zip -c flag.txt -p flagf.txt
```
The flags ```-C``` and ```-c``` tells us the file we want to match with the known plaintext is the file ```flag.txt``` in ```secret-files.zip``` and ```-p``` to specify the known plaintext.  

This takes a while to run, but eventually tells us the key candidates
```bash
Generated 4194304 Z values.
[19:47:27] Z reduction using 4 bytes of known plaintext
100.0 % (4 / 4)
1241014 values remaining.
[19:47:27] Attack on 1241014 Z values at index 7
Keys: f5af793b 6d3ea7ba 9b71082d
12.4 % (154061 / 1241014)
[19:50:53] Keys
f5af793b 6d3ea7ba 9b71082d
```

We can then retrieve the files with the same key candidates:
```bash
$ bkcrack -C secret-files.zip -c flag.txt -k f5af793b 6d3ea7ba 9b71082d -d flag.txt
```
Reading the deciphered file gives ```govtech-csg{EnCrYpT!0n_D0e$_NoT_M3@n_Y0u_aR3_s4f3}```
## Method 2: Indenmity Form
We can see that the other file in ```secret-files.zip``` is the [Consent and Indemnity Form](), which we have access to. Note that we have to compress it the same way to ensure that the known plaintext bytes are the same. We note earlier that this is deflated, so we can compress it in the following manner into a ```test.zip``` file.
``` 
$ zip test.zip STACK\ the\ Flags\ Consent\ and\ Indemnity\ Form.docx
```
We can quickly verify that the [CRC](https://en.wikipedia.org/wiki/Cyclic_redundancy_check) values are the same with
```
$ zipinfo -v file
```
where we substitute ```file``` with ```secret-files.zip``` and ```test.zip```. 
With this out of the way, we can finally start our attack!
```bash
$ bkcrack -C secret-files.zip -c STACK\ the\ Flags\ Consent\ and\ Indemnity\ Form.docx -P test.zip -p STACK\ the\ Flags\ Consent\ and\ Indemnity\ Form.docx
```
The flag ```-C``` and ```-c``` tells us the file we want to match with the known plaintext is the file ```STACK the Flags Consent and Indemnity Form.docx``` in ```secret-files.zip``` and ```-P``` and ```-p``` to specify the zip file with the known plaintext respectively.  

This runs much faster than the previous method since there are more bytes of known plaintext and we obtain
```bash
Generated 4194304 Z values.
[20:15:01] Z reduction using 273302 bytes of known plaintext
9.7 % (26568 / 273302)
74 values remaining.
[20:15:06] Attack on 74 Z values at index 247036
Keys: f5af793b 6d3ea7ba 9b71082d 
100.0 % (74 / 74)
[20:15:06] Keys
f5af793b 6d3ea7ba 9b71082d 
```

We can then retrieve the files with the same key candidates:
```bash
$ bkcrack -C secret-files.zip -c flag.txt -k f5af793b 6d3ea7ba 9b71082d -d actualflag.txt
```


Reading the deciphered file gives ```govtech-csg{EnCrYpT!0n_D0e$_NoT_M3@n_Y0u_aR3_s4f3}```

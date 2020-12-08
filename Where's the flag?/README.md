# Where's the flag?

### Description
**_There's plenty of space to hide flags in our spacious office. Let's see if you can find it!_**

### Solution

There is a png file provided that shows a photo of the GovTech cafeteria. 

<img src="misc-challenge-7.png">

With any Stego challenge, it is usually a good idea to run **file** and **strings** on the target file. 

<img src="screenshot1.PNG">

We don't get anything interesting from the results. Next, we ran **pngcheck** on the file, which checks for the integrity of PNG data chunks.

<img src="screenshot2.PNG">

Hmmm, we get an error saying that the zTXt keyword is longer than the limit. zTXt chunks are generally used for conveying textual information about the image which might contain the flag. We can extract the zTXt chunk through a hex editor or by uploading it to an online extractor like [this](https://www.dcode.fr/png-chunks).

The extracted data resembles Base64 encoded data. We upload it to [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)) to decode it.

Bingo, the decodes data contains the PNG header. We save it as a PNG file and find that it contains a picture of an actual flag.

<img src="flag.png">



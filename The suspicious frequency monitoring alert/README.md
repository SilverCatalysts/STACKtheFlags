# The suspicious frequency monitoring alert!

## Description
We received an alert from our smart city’s frequency monitoring and noticed some anomalies. Figure out what is happening

## Solving
The challenge involved a pcap file, which essentially captures live network packet data. Given the title of the problem, we know to look out for any suspicious data transmitted.

To view the data, we ran ```strings iot-challenge-2.pcap > frequency.txt```.

Now, looking through ```frequency.txt```, we see many wifi names as well as the data transmitted.

However, for one wifi, we notice a base64 string being transmitted.

![sus](/sus.png "Suspicious Frequency")

Decoding the string **Z292dGVjaC1j**, we see that it decodes to **govtech-c**, which represents the start of the flag.

Now that we know the network **TP-Link_1491_0** is transmitting suspicious data, we can find all the data it transmitted by simply running ```grep -A1 "TP-Link_1491_0" frequency.txt```. 

This will look for all the occurances of **TP-Link_1491_0** as well as the transmitted data, which is the line below it.

We get

![parts](/parts.png "Parts of the flag")

Alternatively, noticing that the first part of the flag starts with **1:**, we can find the remaining parts by grepping **2:**, **3:** and **4:**.

Next, we concatenate all the parts to get **Z292dGVjaC1jc2d7RXhmaWx0UkFUaW9OX1dpRmlfSU9UIX0=**.

Decoding this, we get **govtech-csg{ExfiltRATioN_WiFi_IOT!}**, which is our flag!

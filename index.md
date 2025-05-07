# Welcome to the FreePBX AI assistant project!

This project intent is to create an extension programable on buisness IP Phones, linked to an OpenAI asssitant. Useful for buisnesses, schools or research facilities, its entent is to quickly answer a question without the need to search the Web. Integrated into a phone, it functions similarly to an consummer available voice assistant (Google, Siri, Alexa, ...). However, it does not listen permanently and answer on a simple phone call. 

In this tutorial, you can follow step by step on the creation of a virtual extension on FreePBX, a python executable code and some custom sounds utilization. 

---

## 1.Requirements

This project is based around the usage of an self hosted IP PBX. For more information on what an IP PBXis, please visit this [Wikipedia](https://en.wikipedia.org/wiki/IP_PBX) page. Please visit the [FreePBX](https://www.freepbx.org/get-started/) for proper install. The phone provisonning server (in this case a virtual machine), contains all the necessary scripts, sounds and files to make the project possible. 
In such hosted system, an IP phone is necessary to communicate with the provisonning server. This project utilizes a [Polycom VVX 411](https://www.voipsupply.com/polycom-vvx-411) IP phone which has the advantage of high quality communication. 
<pre><code>install IPBX</code></pre>

---

## 2.Installing the necessary software


# Welcome to the FreePBX AI assistant project!

This project intent is to create an extension programable on buisness IP Phones, linked to an OpenAI asssitant. Useful for buisnesses, schools or research facilities, its entent is to quickly answer a question without the need to search the Web. Integrated into a phone, it functions similarly to an consummer available voice assistant (Google, Siri, Alexa, ...). However, it does not listen permanently and answer on a simple phone call. 

In this tutorial, you can follow step by step on the creation of a virtual extension on FreePBX, a python executable code and some custom sounds utilization. 

---

## 1.Requirements

This project is based around the usage of an self hosted IP PBX. For more information on what an IP PBX is, please visit this [Wikipedia](https://en.wikipedia.org/wiki/IP_PBX) page. Please visit the [FreePBX](https://www.freepbx.org/get-started/) website for proper install. The phone provisioning server (in this case a virtual machine), contains all the necessary scripts, sounds and files to make the project possible. 
In such hosted system, an IP phone is necessary to communicate with the provisioning server. This project utilizes a [Polycom VVX 411](https://www.voipsupply.com/polycom-vvx-411) IP phone which has the advantage of high quality communication. 
<pre><code>install IPBX</code></pre>

---

## 2.Installing the necessary software

Once the operating system is installed, please note down the IP adress of the provisioning server, so it can be accessed from any device of your home. 
The linux terminal in Debian is used to install the necessary software:
- [Python](https://wiki.debian.org/Python)
- [Google Cloud services API](https://github.com/googleapis/google-api-python-client) for which you need to create a service account key
- [OpenAI cloud API](https://platform.openai.com/docs/libraries?language=python) for which you need to create an API key
- [pydub](https://pypi.org/project/pydub/)
- [subprocess](https://pypi.org/project/subprocess.run/)

💡 **Note:** Always use the root user in the Linux terminal by running first the ``su`` command and entering your root password setup during installation. 

---

## 3.Create the virtual extension

Log in your FreePBX dashboard using the IP adress noted at the begining of section 2. 

After your phones had been setup with an extension in the menu Connectivity>Extensions, they should be up and running and able to call themselves. To call the AI assistant, a virtual extension need to be created (in our case extension 8008) than can be called on demand by each person owning a phone. 

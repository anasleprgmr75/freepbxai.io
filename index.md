# Welcome to the FreePBX AI assistant project!

This project intent is to create an extension programable on buisness IP Phones, linked to an OpenAI asssitant. Useful for buisnesses, schools or research facilities, its entent is to quickly answer a question without the need to search the Web. Integrated into a phone, it functions similarly to an consummer available voice assistant (Google, Siri, Alexa, ...). However, it does not listen permanently and answer on a simple phone call. 

In this tutorial, you can follow step by step on the creation of a virtual extension on FreePBX, a python executable code and some custom sounds utilization. 

---

## 1. Requirements

This project is based around the usage of an self hosted IP PBX. For more information on what an IP PBX is, please visit this [Wikipedia](https://en.wikipedia.org/wiki/IP_PBX) page. Please visit the [FreePBX](https://www.freepbx.org/get-started/) website for proper install. The phone provisioning server (in this case a virtual machine), contains all the necessary scripts, sounds and files to make the project possible. 
In such hosted system, an IP phone is necessary to communicate with the provisioning server. This project utilizes a [Polycom VVX 411](https://www.voipsupply.com/polycom-vvx-411) IP phone which has the advantage of high quality communication. 

---

## 2. Installing the necessary software

Once the operating system is installed, please note down the IP adress of the provisioning server, so it can be accessed from any device of your home. 
The linux terminal in Debian is used to install the necessary software:
- [Python](https://wiki.debian.org/Python)
- [Google Cloud services API](https://github.com/googleapis/google-api-python-client) for which you need to create a service account key
- [OpenAI cloud API](https://platform.openai.com/docs/libraries?language=python) for which you need to create an API key, enable the STT API and the Gemini generative AI API
- [pydub](https://pypi.org/project/pydub/)
- [subprocess](https://pypi.org/project/subprocess.run/)

💡 **Note:** Always use the root user in the Linux terminal by running first the ``su`` command and entering your root password setup during installation. 

---

## 3. Create the virtual extension

Log in your FreePBX dashboard using the IP adress noted at the begining of section 2. 

After your phones had been setup with an extension in the menu Connectivity>Extensions, they should be up and running and able to call themselves. To call the AI assistant, a virtual extension need to be created (in our case extension 8008) than can be called on demand by each person owning a phone. 

💡 **Note:** Feel free to use any extension number you desire, keeping in mind the already existing reserved extension numbers and dialcodes (voicemail, wake-up call, ...)

### 3.1 Modify custom_extensions.conf

Open the Linux terminal and edit the ``custom_extensions.conf`` file with the following command:
<pre><code>sudo nano /etc/asterisk/extensions_custom.conf</code></pre>
For now, only add the following lines:
<pre><code>[custom-openai]
  exten => 8008,1,Answer()
  same => n,Wait(1)
  same => n,Hangup()
</code></pre>

This file contains what is called the dialplan of extension 8008, or in other words, what your FreePBX server will execute when you dial extension 8008. 
You may then reload the dialplan with <pre><code>sudo asterisk -rx "dialplan reload"</code></pre>

### 3.2 Add a custom destination in FreePBX 

On your FreePBX config webpage, go to Admin > Custom Destinations. Add a new one:
Custom Destination: custom-openai,8008,1
Description: ChatGPT Assistant

Submit and Apply Config
### 3.3 Create a Misc Application
Go to Applications > Misc Applications and add :
Feature Code: 8008
Destination>Custom Destination : ChatGPT Assistant

Submit and Apply Config

❗ **Note:** When modifying a FreePBX file, always reload the dialplan for the configuration to take place. Same applies when modifying from the user interface in 3.2 and 3.3, submit first and then click on the red square in the upper right corner "Apply Config". 


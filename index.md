# Welcome to the FreePBX AI assistant project!

This project's intent is to create an extension programmable on business IP Phones, linked to an OpenAI assitant. Useful for businesses, schools or research facilities, its intent is to quickly answer a question without the need to search the Web. Integrated into a phone, it functions similarly to a consumer available voice assistant (Google, Siri, Alexa, ...). However, it does not listen permanently and answer on a simple phone call.

In this tutorial, you can follow step by step the creation of a virtual extension on FreePBX, a python executable code and some custom sounds utilization. 
Consult this [Video](https://anasleprgmr75.github.io/freepbxai.io/demo.mp4)

---

## 1. Requirements and Introduction

This project is based around the usage of a self hosted IP PBX. For more information on what an IP PBX is, please visit this [Wikipedia](https://en.wikipedia.org/wiki/IP_PBX) page. The phone provisioning server (in this case a virtual machine), contains all the necessary scripts, sounds and files to make the project possible. 
With such hosted system, an IP phone is necessary to communicate with the provisioning server. This project utilizes a [Polycom VVX 411](https://www.voipsupply.com/polycom-vvx-411) IP phone which has the advantage of HD sound communication. 

The script is intended to follow these steps : create a recordig of the user's voice, submit it to Google cloud for STT to obtain the transcript, submit the transcript to Google Cloud again for generative AI response, submit the response to OpenAI TTS, receive the file and last but not least, play it to the user. 
Google cloud was used for STT and generative text since they are quite inexpensive or free for small usage like ours. OpenAI was used for TTS since their voice models are realistic and bring a human touch to the answer. 

---

## 2. Installing the necessary software

Once the Debian operating system is installed, please note down the IP adress of the computer, so it can be accessed later from any device of your home. 
The linux terminal in Debian is used to install the necessary software:
- [FreePBX](https://www.freepbx.org/get-started/)
- [Python](https://wiki.debian.org/Python)
- [Google Cloud services API](https://github.com/googleapis/google-api-python-client) for which you need to create a service account key, enable the STT API and the Gemini generative AI API
- [OpenAI cloud API](https://platform.openai.com/docs/libraries?language=python) for which you need to create an API key,
- [pydub](https://pypi.org/project/pydub/)
- [subprocess](https://pypi.org/project/subprocess.run/)

💡 **Note:** Always use the root user in the Linux terminal by running first the ``su`` command and entering your root password, during installation. 

---

## 3. Create the virtual extension

Log in your FreePBX dashboard using the IP adress noted at the begining of section 2. 

After your phones had been setup with an extension in the menu Connectivity>Extensions, they should be up and running and able to call themselves. To call the AI assistant, a virtual extension needs to be created (in our case extension 8008) than can be called on demand by each person owning a phone. 

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
You may then reload the dialplan with ``sudo asterisk -rx "dialplan reload"``

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

---

## 4. Create the recording

💡 **Note:** Troubleshooting and debugging can be done from the FreePBX user interface in Reports>System Logfiles. 

After python code experimenting, a successful case scenario was obtained when a recording of the user question is first created before executing the Python code. To perform this, the ``custom_extensions.conf`` file can be modified to create a recording in the dialplan with ``sudo nano /etc/asterisk/extensions_custom.conf``. 

There, the following lines can be added :

<pre><code>[custom-openai]
  exten => 8008,1,Answer()
  same => n,Wait(1)
  same => n,Playback(beep)
  same => n,MixMonitor(/var/lib/asterisk/sounds/custom/recording.wav)
  same => n,WaitForSilence(2000 200)
  same => n,StopMixMonitor()
  same => n,AGI(script.py)
  same => n,Playback(/var/lib/asterisk/sounds/custom/finalplayback)
  same => n,Hangup()
</code></pre>

A few explanations for this code : the function ``Playback`` plays a desired sound to the phone user. The beep notifies the user it is allowed to talk, the ``MixMonitor`` function records the phone line and saves it as a ``wav`` file in a custom directory. When silence is detected on the line for 2s, the recording stops and the python script can be run with the ``AGI``command. Lastly, when the script has generated a playback file received from OpenAI, it is played to the user. 

The python script does not exist yet and will be created in the next section. You may dial the phone at this point while looking in the log files of asterisk to check if you hear a beep and then see a recording appear in ``/var/lib/asterisk/sounds/custom``.

✅ **Info:** The ``Playback`` function does not require a ``wav`` or ``ulaw`` extension while the ``MixMonitor`` function does.

❗ **Note:** Do not forget the ``sudo asterisk -rx "dialplan reload"`` command before dialing. 


---

## 5. Create the python executable code

💡 **Note:** Troubleshooting and debugging can be done from the FreePBX user interface in Reports>System Logfiles. Considering the length of the python script, it is recommended to first work in Visual Studio Code to test the different libraries and test each bit of code in the AGI script execution by dialing the number. This method is long and tedious, but allows for smooth coding and rapid isolation of issues. Indeed, while Visual Studio Code displays the errors during execution, FreePBX does not give details on the specific line that caused the code to crash. 

The asterisk system, at the heart of the FreePBX software has access to files in its directories. This unfortunately means that the python code needs to be modified within the Linux terminal in the ``su`` mode. 

Open (or create) the file script.py by typing ``sudo nano /var/lib/asterisk/agi-bin/script.py``. And add the following first lines: 

```python
#!/usr/bin/python
import sys
from google.cloud import speech
from google.oauth2 import service_account
import google.generativeai as genai
from openai import OpenAI
from pydub import AudioSegment
import subprocess

```
❗ **Note:** The first line of this code is very important, and indicates the ``AGI`` command that the file is written in python. The rest of the libraries are for future use in the code. 

The following lines in the code are for communication between the python file and Asterisk. 
```python
def agi_command(cmd):
    sys.stdout.write(cmd + '\n')
    sys.stdout.flush()
    return sys.stdin.readline().strip()
```
This function translates a python command to execute it inside the Asterisk command line such as ``Playback``, ``MixMonitor`` and even ``Verbose`` to print messages in the Asterisk log files. 

The following lines are for authentification purposes to the Google Cloud and OpenAI APIs. 

```python
#APIs identifications
client_file = "/var/lib/asterisk/agi-bin/google-creds.json"
credentials = service_account.Credentials.from_service_account_file(client_file)
client = speech.SpeechClient(credentials=credentials)
genai.configure(credentials=credentials)
agi_command("VERBOSE \"Authentification done\" 2")
```
Note the location of the google json service account file, that can be downloaded from Google cloud. The ``VERBOSE`` command is created for troubleshooting reasons, to display a message in the Asterisk Logfiles. 

Then add the folowing lines responsible for sending the recording to Google Cloud STT. 

```python
#Sending audio to google for STT
agi_command("VERBOSE \"Sending file to google\" 2")
import io
with io.open("/var/lib/asterisk/sounds/custom/recording.wav", 'rb') as f: 
    content = f.read()
    audio = speech.RecognitionAudio(content=content)
config = speech.RecognitionConfig(language_code = 'en-US')
question = client.recognize(config=config, audio = audio)
transcript = question.results[0].alternatives[0].transcript
```
And then creating a generative AI request to Google Cloud

```python
#Generating text from Google STT with Gemini
model = genai.GenerativeModel("gemini-2.0-flash")
chat = model.start_chat(history=[
    {"role": "user", 
        "parts": ["You are a friendly phone assistant and directly give answer without asking question. Sound human and always say goodbye."]}])
response = chat.send_message(str(question))
```
The model ``gemini-2.0-flash`` was chosen for this project, but feel free to choose one that suits your needs. The ``parts`` parameter describes the general behaviour of the response requested from Google Cloud Gemini. 

💡 **Note:** In a more developped version of this code, it would possible to integrate the question with a user history and personalize even more the response obtained from Google Cloud. A longer, more detailed response can cost money when it comes to the next steps with an OpenAI TTS. 

Now, with an obtained response from Google Cloud, the TTS engine can be utilized to create a human sounding voice for the user.

```python
#Generating voice with OpenAI TTS
speechclient = OpenAI(api_key="YOUR API KEY FROM OPENAI")
audio = speechclient.audio.speech.create(
    model = "tts-1",
    voice = 'nova',
    input=str(response.text))
audio.stream_to_file("/var/lib/asterisk/sounds/custom/speech.wav")

```
The ``tts-1`` model is used since audio quality will be changed in the next step. Feel free to use any other voice than ``nova``. Visit the [OpenAI documentation](https://platform.openai.com/docs/guides/text-to-speech) for personalization of the audio file, its emotions and its speed. 

From practice, it was noticed that Asterisk and FreePBX have very specific requirements when it comes to the encoding in ``wav`` files. The more supported ``ulaw`` file format conserves the audio quality and allows for Asterisk playback. Within Python, it is possible to convert the ``wav`` file into a ``ulaw`` file with the following lines. 

```python
#Convert OpenAI WAV to ULAW for playback in Asterisk
subprocess.run([
    "ffmpeg",
    "-i", "/var/lib/asterisk/sounds/custom/speech.wav",
    "-ar", "8000",
    "-ac", "1",
    "-af", "volume=6dB",
    "-acodec", "pcm_mulaw",
    "-f", "mulaw",
    "-y",
    "/var/lib/asterisk/sounds/custom/finalplayback.ulaw"])
```
Note a slight volume adjustment with ``volume=6dB``. 

Now that the code is finished, do not the forget the following line to make the python code executable by Asterisk:
<pre><code>sudo chmod +x /var/lib/asterisk/agi-bin/script.py</code></pre>
And of course, update the dialplan!
<pre><code>sudo asterisk -rx "dialplan reload"</code></pre>

---

## 6. Tweaking the virtual extension

Custom sounds and audio files can be created by visiting [OpenAI FM playground](https://www.openai.fm/) for free! You can create for instance a welcome sound to great the user and give basic instructions ! Once downloaded, the files can be converted usding the folowing script and copied to ``/var/lib/asterisk/sounds/custom`` for a nicer experience. 

```python
from pydub import AudioSegment
import subprocess

subprocess.run([
    "ffmpeg",
    "-i", "YOUR_INPUT_FILE.wav",
    "-ar", "8000",         
    "-ac", "1",           
    "-af", "volume=2dB",
    "-acodec", "pcm_mulaw", 
    "-f", "mulaw",        
    "-y",              
    "YOUR_OUTPUT_FILE.ulaw"])
```
Many other parameters or types of API can be implemented in a FreePBX system, the possibilities are endless! A SIP trunk with a phone number can also be used for calling the AI anywhere you are. 

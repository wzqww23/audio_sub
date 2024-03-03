# Auto Synced & Translated Dubs
 Automatically translates the text of a video into chosen languages based on a subtitle file, and also uses AI voice to dub the video, while keeping it properly synced to the original video using the subtitle's timings.
 
### How It Works
If you already have a human-made SRT subtitles file for a video, this will:
1. Use Google Cloud/DeepL to automatically translate the text, and create new translated SRT files
2. Use the timings of the subtitle lines to calculate the correct duration of each spoken audio clip
3. Create text-to-speech audio clips of the translated text (using more realistic neural voices)
4. Stretch or shrink the translated audio clip to be exactly the same length as the original speech.
    - Optional (On by Default): Instead of stretching the audio clips, you can instead do a second pass at synthesizing each clip through the API using the proper speaking speed calculated during the first pass. This slightly improves audio quality.
    - If using Azure TTS, this entire step is not necessary because it allows specifying the desired duration of the speech before synthesis
5. Builds the audio track by inserting the new audio clips at their correct time points. Therefore the translated speech will remain perfectly in sync with the original video.
    
### More Key Features
- Creates translated versions of the SRT subtitle file
- Batch processing of multiple languages in sequence
- Config files to save translation, synthesis, and language settings for re-use
- Allows detailed control over how the text is translated and synthesized
   - Including: A "Don't Translate" phrase list, a manual translation list, a phoneme pronunciation list, and more

### Additional Included Tools
- `TrackAdder.py`: Adds all language audio tracks to a video file
   - With ability to merge a sound effects track into each language track
- `TitleTranslator.py`: Translates a YouTube video Title and Description to multiple languages
- `TitleDescriptionUpdater.py`: Uses YouTube API to update the localized titles and descriptions for a YouTube video using output of TitleTranslator.py
- `SubtitleTrackRemover.py`: Uses YouTube API to remove a specific audio track from a YouTube video
- `TranscriptTranslator.py`: Translates an entire transcript of text
- `TranscriptAutoSyncUploader.py`: Using YouTube API, it lets you upload a transcript for a video, then have YouTube sync the text to the video
   - You can also upload multiple pre-translated transcripts and have YouTube sync it, assuming the language is supported
- `YouTube_Synced_Translations_Downloader.py`: Using YouTube API, translate the captions of a video into the specified languages, then download the auto-synced subtitle file created by YouTube
----

# Instructions

### External Requirements:
- ffmpeg must be installed (https://ffmpeg.org/download.html)

### Optional External Tools:
- Optional: Instead of ffmpeg for audio stretching, you could use the program'rubberband'
  - I've actually found ffmpeg works better, but I'll still leave the option for rubberband if you want.
  - If using Rubberband, yoou'll need the rubberband binaries. Specifically on [this page]((https://breakfastquay.com/rubberband/), find the download link for "Rubber Band Library v3.3.0 command-line utility" (Pick the Windows or MacOS version depending). Then extract the archive to find:
     - On Windows: rubberband.exe, rubberband-r3.exe, and sndfile.dll
     - On MacOS: rubberband, rubberband-r3
  - Doesn't need to be installed, just put the above mentioned files in the same directory as main.py

## Setup & Configuration
1. Download or clone the repo and install the requirements using `pip install -r requirements.txt`
   - I wrote this using Python 3.9 but it will probably work with earlier versions too
2. Install the programs mentioned in the 'External Requirements' above.
3. Setup your Google Cloud (See Wiki), Microsoft Azure API access and/or DeepL API Token, and set the variables in `cloud_service_settings.ini`. 
   - I recommend Azure for the TTS voice synthesizing because they have newer and better voices in my opinion, and in higher quality (Azure supports sample rate up to 48KHz vs 24KHz with Google). 
   - Google Cloud is faster, cheaper and supports more languages for text translation, but you can also use DeepL.
4. Set up your configuration settings in `config.ini`. The default settings should work in most cases, but read through them especially if you are using Azure for TTS because there are more applicable options you may want to customize.
   - This config includes options such as the ability to skip text translation, setting formats and sample rate, and using two-pass voice synthesizing
5. Finally open `batch.ini` to set the language and voice settings that will be used for each run. 
   - In the top `[SETTINGS]` section you will enter the path to the original video file (used to get the correct audio length), and the original subtitle file path
   - Also you can use the `enabled_languages` variable to list all the languages that will be translated and synthesized at once. The numbers will correspond to the `[LANGUAGE-#]` sections in the same config file. The program will process only the languages listed in this variable.
   - This lets you add as many language presets as you want (such as the preferred voice per language), and can choose which languages you want to use (or not use) for any given run.
   - Make sure to check supported languages and voices for each service in their respective documentation.

## Usage Instructions
- **How to Run:** After configuring the config files, simply run the main.py script using `python main.py` and let it run to completion
   - Resulting translated subtitle files and dubbed audio tracks will be placed in a folder called 'output'
- **Optional:** You can use the separate `TrackAdder.py` script to automatically add the resulting language tracks to an mp4 video file. Requires ffmpeg to be installed.
   - Open the script file with a text editor and change the values in the "User Settings" section at the top.
   - This will label the tracks so the video file is ready to be uploaded to YouTube. HOWEVER, the multiple audio tracks feature is only available to a limited number of channels. You will most likely need to contact YouTube creator support to ask for access, but there is no guarantee they will grant it.
- **Optional:** You can use the separate `TitleTranslator.py` script if uploading to YouTube, which lets you enter a video's Title and Description, and the text will be translated into all the languages enabled in `batch.ini`. They wil be placed together in a single text file in the "output" folder.

----

## Additional Notes:
- This works best with subtitles that do not remove gaps between sentences and lines.
- For now the process only assumes there is one speaker. However, if you can make separate SRT files for each speaker, you could generate each TTS track separately using different voices, then combine them afterwards.
- It supports both Google Translate API and DeepL for text translation, and Google, Azure, and Eleven Labs for Text-To-Speech with neural voices.
- This script was written with my own personal workflow in mind. That is:
    - I use [**OpenAI Whisper**](https://github.com/openai/whisper) to transcribe the videos locally, then use [**Descript**](https://www.descript.com/) to sync that transcription and touch it up with corrections.
    - Then I export the SRT file with Descript, which is ideal because it does not just butt the start and end times of each subtitle line next to each other. This means the resulting dub will preserve the pauses between sentences from the original speech. If you use subtitles from another program, you might find the pauses between lines are too short.
    - The SRT export settings in Descript that seem to work decently for dubbing are *150 max characters per line*, and *1 max line per card*.
- The "Two Pass" synthesizing feature (can be enabled in the config) will drastically improve the quality of the final result, but will require synthesizing each clip twice, therefore doubling any API costs.

### Currently Supported Text-To-Speech Services:
- Microsoft Azure
- Google Cloud
- Eleven Labs

### Currently Supported Translation Services:
- Google Translate
- DeepL

## For more information on supported languages by service:
- [Google Cloud Translation Supported Languages](https://cloud.google.com/translate/docs/languages)
- [Google Cloud Text-to-Speech Supported Languages](https://cloud.google.com/text-to-speech/docs/voices)
- [Azure Text-to-Speech Supported Languages](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/language-support#text-to-speech)
- [DeepL Supported Languages](https://www.deepl.com/docs-api/translating-text/request/)

----

### For Result Examples See: [Examples Wiki Page](https://github.com/ThioJoe/Auto-Synced-Translated-Dubs/wiki/Examples)
### For Planned Features See: [Planned Features Wiki Page](https://github.com/ThioJoe/Auto-Synced-Translated-Dubs/wiki/Planned-Features)
### For Google Cloud Project Setup Instructions See: [Instructions Wiki Page](https://github.com/ThioJoe/Auto-Synced-Translated-Dubs/wiki/Instructions:-Obtaining-an-API-Key)
### For Microsoft Azure Setup Instructions See: [Azure Instructions Wiki Page](https://github.com/ThioJoe/Auto-Synced-Translated-Dubs/wiki/Instructions:-Microsoft-Azure-Setup)


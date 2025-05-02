---
link: https://blog.devops.dev/f5-text-to-speech-f5-tts-a-compact-and-efficient-voice-cloning-model-to-run-on-your-local-536c292d5e1d
byline: Carlos Biagolini-Jr.
site: DevOps.dev
date: 2025-03-06T12:05
excerpt: F5 Text-to-Speech (F5-TTS) is a powerful open-source tool that converts written text into high-quality, lifelike audio. What makes it particularly impressive is its voice cloning capability, which…
twitter: https://twitter.com/@devops_blog
slurped: 2025-03-08T07:41
title: "F5 Text-to-Speech (F5-TTS): A Compact and Efficient Voice Cloning Model to Run on Your Local Machine"
tags:
  - text-to-speech
---
## Introduction

F5 Text-to-Speech (F5-TTS) is a powerful open-source tool that converts written text into high-quality, lifelike audio. What makes it particularly impressive is its **voice cloning capability**, which requires only a small sample of audio to accurately replicate a voice — a feature that, as of today, is remarkably advanced. This makes it an excellent choice for users looking to clone voices with minimal data, which is ideal for applications such as personalized voiceovers, virtual assistants, and accessibility tools.

However, if your goal is purely text-to-speech (TTS) generation without the need for cloning, other tools might be more suitable. There are **commercially established alternatives**, such as [**ElevenLabs**](https://elevenlabs.io/app), which offer a high-quality and widely used voice synthesis service, though with an associated cost. On the other hand, there are **local-run alternatives**, like [**Kokoro TTS**](https://huggingface.co/hexgrad/Kokoro-82M), which provide satisfactory quality without relying on online services, allowing greater control over the processing.

If you’re interested in exploring different options, I have tested various TTS models and shared my insights in other posts — check them out at [my Medium page](https://medium.com/@biagolini).

## Key Features:

- **High-quality audio synthesis** using pre-trained models.
- **Customizable settings** for speed, pitch, and denoising.
- **Web interface** for easy usage through Gradio.
- Compatibility with **GPU-accelerated environments** for efficient processing.

## Resources

To set up the F5-TTS tool, you can refer to the following resources for its code, pre-trained models, and documentation:

- **GitHub repository**: [F5-TTS GitHub](https://github.com/swivid/F5-TTS/)
- **Hugging Face model**: [F5-TTS on Hugging Face](https://huggingface.co/SWivid/F5-TTS)
- **Hugging Face test space**: [Try it here](https://huggingface.co/spaces/mrfakename/E2-F5-TTS)
- **Scientific article**: [PDF link](https://arxiv.org/pdf/2410.06885)

## Tested Environments

- **Operating System**: Ubuntu 24.04 via WSL (Windows Subsystem for Linux)
- **Python Version**: `Python 3.12.4`
- **GPU Support**: CUDA Toolkit 12.6 for NVIDIA GPUs. [Download CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit)
- **Audio Processing Tools**: `ffmpeg` for handling audio files.

## Setting Up F5-TTS: Step-by-Step

Here, I present instructions for setting up and running F5-TTS on your local machine.

## Step 1: Create a Virtual Environment

Using a virtual environment ensures that dependencies are isolated and managed effectively. Follow these steps:

mkdir -p <your-project-path>  
cd <your-project-path>  
python -m venv venv  
source venv/bin/activate  
pip install --upgrade pip

## Step 2: Install PyTorch

Choose the installation method based on your GPU type:

For NVIDIA GPUs:

pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124

For AMD GPUs:

pip install torch==2.5.1+rocm6.2 torchaudio==2.5.1+rocm6.2 --extra-index-url https://download.pytorch.org/whl/rocm6.2

## Step 3: Install PortAudio

Update your system and install the required packages:

sudo apt update -y  
sudo apt install ffmpeg portaudio19-dev  -y

## Step 4: Install F5-TTS

Install F5-TTS directly from its GitHub repository:

pip install git+https://github.com/SWivid/F5-TTS.git  
pip list  # Verify installed dependencies

## Step 5: Launch the Tool

Run the F5-TTS Gradio web interface:

f5-tts_infer-gradio --port 7860 --host 0.0.0.0

Once launched, open your browser and navigate to [http://localhost:7860](http://localhost:7860/) to access the tool.

## Using F5-TTS: Basic Voice Cloning and Synthesizing Text to High-Quality Audio

In this section, we’ll explore the simplest way to use F5-TTS for voice cloning and text-to-speech synthesis.

## Step 1: Clone a Voice

On the main interface of the application, ensure you are in the `Basic TTS` tab. Next, upload a reference audio file by clicking the appropriate upload button. This audio will serve as the basis for voice cloning.

## Step 2: Define the Text to Synthesize

In the `Text to Generate` section, input the text you want to synthesize.  
For example:

The gentle symphony of nature unfolds in every corner of our world.

## Step 3: Synthesize

To generate the audio, click **Synthesize**. After processing, listen to the output and make adjustments if necessary.

If you’re satisfied with the result, use the **Download** button to save the audio file.

## Optional: Advanced Configuration

If you want to refine the audio further, you can experiment with the following advanced settings:

- **Speed**: Adjust the playback speed of the synthesized voice.
- **NFE Steps**: Fine-tune noise filtering for clearer audio.
- **Silence Removal**: Remove pauses or silences, especially useful for longer text inputs.

## Using F5-TTS: Advanced Voice Cloning and Synthesizing Text to High-Quality Audio

In this section, we will explore the advanced method of voice cloning. This approach allows you to assign specific emotional tones to segments of speech, enabling the generation of varied and expressive audio outputs.

The system can synthesize speech using the designated emotional tone. If no tone is specified, the model defaults to the **Regular** speech type. Each specified tone will remain in use until another is indicated.

## Step 1: Prepare Reference Audio

Before synthesizing speech with emotional tones, you’ll need to record multiple audio samples with different vocal intonations to use as references. For instance, create recordings for each tone with the suggested content below. Save each sentence as a separate file:

{Regular} Hello, I'd like to order a sandwich please.  
{Surprised} What do you mean you're out of bread?  
{Sad} I really wanted a sandwich though...

## Step 2: Upload Each Audio File

In the application interface, navigate to the `Multiple Speech` tab.

For each reference audio:

- Add the emotional tone name in the **Speech Type Name** field (e.g., `Regular`, `Surprised`, `Sad`).
- Upload the corresponding file in the **Reference Audio** field.
- Enter the spoken content from the reference audio in the **Reference Text** field.
- Click **Add Speech Type** to save this tone configuration.

Repeat this process for all your audio samples.

## Step 3: Define Text to Generate

In the `Text to Generate` section, input the text you wish to synthesize. Unlike simple text-to-speech generation, here you can include emotional tone tags to indicate the desired tone for each segment. If no tag is provided, the system will use the **Regular** tone by default.

For example:

{Regular} Voice cloning technology has advanced significantly in recent years, allowing us to replicate human speech with astonishing accuracy.  
{Surprised} Wait, are you saying this isn't a real person speaking right now?  
{Sad} It's incredible... but also a little unsettling to think that voices can be copied so perfectly.

## Step 4: Synthesize

To generate the audio, click **Synthesize**. After processing, listen to the output and refine the text or tones as needed.

Once satisfied, use the **Download** button to save the final audio file.

## Conclusion

F5-TTS is a remarkable tool that makes high-quality text-to-speech synthesis, including voice cloning, accessible to everyone. By following the steps outlined in this guide, you can set up the tool and experiment with its capabilities, from simple text generation to advanced emotional voice cloning.

Its flexibility and robust performance make it suitable for a wide range of applications, including entertainment, education, accessibility, and more.

Take the leap into the world of advanced TTS synthesis and explore the creative possibilities with F5-TTS!
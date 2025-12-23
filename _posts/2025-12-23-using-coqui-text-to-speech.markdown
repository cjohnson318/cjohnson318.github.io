---
layout: post
title: "Using Coqui's Text To Speech"
date: 2025-12-23 00:00:00 -0700
tags: python
---

This project uses short .WAV files to convert text to a version of your own
voice. I pulled this together in one evening. Even using Google Gemini for
triaging bugs, the hardest part was just getting the dependencies to work
together. The trick was to set the Python version, then set an environment
variable so that PyTorch's torchdodec could find and use ffmpeg.

## Initial Setup

Set up the project with uv and Python 3.11.

{% highlight console %}
uv init --python 3.11 .
uv add coqui-tts torchcodec
{% endhighlight %}

I created reference voice data with QuickTime, and then converted them from
.M4A to .WAV.

{% highlight console %}
afconvert -f WAVE -d LEI16 input.m4a output.wav
{% endhighlight %}

I needed to install ffmpeg and espeak with brew.

{% highlight console %}
brew install espeak ffmpeg 
{% endhighlight %}

I needed to set this environment variable to get torchcodec working.

{% highlight console %}
export DYLD_LIBRARY_PATH="$(brew --prefix ffmpeg)/lib:$DYLD_LIBRARY_PATH"
{% endhighlight %}


## Phonetic Pangram

A phonetic pangram uses every sound in a language, the way a regular pangram
uses every letter in a language, like "The quick brown fox jumps over the lazy
dog", or "Sphinx of black quartz, judge my vow".

I decided to use phonetic pangrams for my reference clips since they are
designed to cover all of the sounds in a English. Here are the ones I used,

  - “With tenure, Suzie’d have all the more leisure for yachting, but her publications are no good.” (for certain US accents and phonological analyses)
  - “Shaw, those twelve beige hooks are joined if I patch a young, gooey mouth.” (perfect for certain accents with the cot-caught merger)
  - “Are those shy Eurasian footwear, cowboy chaps, or jolly earthmoving headgear?” (perfect for certain Received Pronunciation accents)
  - “The beige hue on the waters of the loch impressed all, including the French queen, before she heard that symphony again, just as young Arthur wanted.” (a phonetic, not merely phonemic, pangram. It contains both nasals [m] and [ɱ] (as in ‘symphony’), the fricatives [x] (as in ‘loch’) and [ç] (as in ‘hue’), and the ‘dark L’ [ɫ] (as in ‘all’) – in other words, it contains different allophones.)

I pulled these from [this](https://clagnut.com/blog/2380/#English_phonetic_pangrams) great source of pangrams.


## Code

{% highlight python %}
import sys

from TTS.api import TTS

MODEL = "tts_models/en/jenny/jenny"
MODEL = "tts_models/en/ljspeech/vits"
MODEL = "tts_models/multilingual/multi-dataset/xtts_v2"

references = [
    "reference_01.wav",
    "reference_02.wav",
    "reference_03.wav",
    "reference_04.wav",
]

def main(filename:str):
    # device = "cuda" if torch.cuda.is_available() else "cpu"
    device = "cpu"
    tts = TTS(MODEL).to(device)
    with open(filename, 'r') as fh:
        text = fh.read()
    tts.tts_to_file(
        text=text,
        speaker_wav=references,
        language="en",
        file_path="output.wav",
    )

if __name__ == '__main__':
    filename = sys.argv[1]
    main(filename)
{% endhighlight %}

Then run and hope for the best.

{% highlight bash %}
uv run main.py script.txt
{% endhighlight %}

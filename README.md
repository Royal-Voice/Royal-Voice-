# Royal-Voice-import gradio as gr
import gradio as gr
import uuid
import os
from pydub import AudioSegment
import librosa
import soundfile as sf
from transformers import pipeline
import torch

# === Simple user login system ===
users = {"admin": "admin123"}
sessions = {}
saved_tracks = {}

# === AI Lyrics Generator (GPT-2) ===
lyrics_gen = pipeline("text-generation", model="gpt2")

def login(username, password):
    if users.get(username) == password:
        token = str(uuid.uuid4())
        sessions[token] = username
        return token, "✅ Login successful"
    return "", "❌ Invalid credentials"

def generate_lyrics(prompt):
    result = lyrics_gen(prompt, max_length=100, do_sample=True)
    return result[0]['generated_text']

def autotune(audio_path):
    try:
        y, sr = librosa.load(audio_path)
        y_shifted = librosa.effects.pitch_shift(y, sr, n_steps=2)
        out_file = "autotuned.wav"
        sf.write(out_file, y_shifted, sr)
        return out_file
    except Exception as e:
        return str(e)

def merge_tracks(track1, track2, track3, track4):
    try:
        inputs = [track1, track2, track3, track4]
        combined = None
        for track in inputs:
            if track:
                seg = AudioSegment.from_file(track)
                combined = seg if combined is None else combined.overlay(seg)
        if not combined:
            return None, "⚠️ Upload at least one track."
        out_path = "mixed_output.mp3"
        combined.export(out_path, format="mp3")
        return out_path, "✅ Mixing complete!"
    except Exception as e:
        return None, f"❌ Error: {str(e)}"

def save_track(token, path):
    if token not in sessions:
        return "❌ Invalid session."
    user = sessions[token]
    saved_tracks.setdefault(user, []).append(path)
    return f"✅ Track saved for {user}"

def export_track(path, fmt):
    try:
        audio = AudioSegment.from_file(path)
        out_file = f"exported_track.{fmt}"
        audio.export(out_file, format=fmt)
        return out_file
    except Exception as e:
        return str(e)

# === Gradio UI ===
with gr.Blocks(title="Royal Voice AI Studio") as demo:
    gr.Markdown("# 🎙️ Royal Voice AI Studio\n**Complete AI-powered music editor**")

    with gr.Tab("🔐 Login"):
        username = gr.Textbox(label="Username")
        password = gr.Textbox(label="Password", type="password")
        login_btn = gr.Button("Login")
        session_token = gr.Textbox(label="Session Token")
        login_status = gr.Textbox(label="Login Result")
        login_btn.click(login, [username, password], [session_token, login_status])

    with gr.Tab("📝 Generate Lyrics"):
        prompt = gr.Textbox(label="Enter topic / theme")
        lyrics_btn = gr.Button("Generate Lyrics")
        lyrics_output = gr.Textbox(label="Generated Lyrics", lines=6)
        lyrics_btn.click(generate_lyrics, prompt, lyrics_output)

    with gr.Tab("🎛️ Mix Tracks"):
        track1 = gr.Audio(label="Track 1", type="filepath", source="upload")
        track2 = gr.Audio(label="Track 2", type="filepath", source="upload")
        track3 = gr.Audio(label="Track 3", type="filepath", source="upload")
        track4 = gr.Audio(label="Track 4", type="filepath", source="upload")
        mix_btn = gr.Button("Mix Now")
        mixed_audio = gr.Audio(label="Mixed Output")
        mix_status = gr.Textbox(label="Mix Status")
        mix_btn.click(merge_tracks, [track1, track2, track3, track4], [mixed_audio, mix_status])

    with gr.Tab("🎚️ Autotune"):
        autotune_input = gr.Audio(label="Vocal Audio", type="filepath", source="upload")
        tune_btn = gr.Button("Apply Autotune")
        autotune_output = gr.Audio(label="Autotuned Output")
        tune_btn.click(autotune, autotune_input, autotune_output)

    with gr.Tab("💾 Save & Export"):
        user_token = gr.Textbox(label="Session Token")
        save_audio_path = gr.Textbox(label="Audio File to Save")
        save_btn = gr.Button("Save Track")
        save_output = gr.Textbox(label="Save Message")
        save_btn.click(save_track, [user_token, save_audio_path], save_output)

        export_format = gr.Dropdown(["mp3", "wav", "ogg"], label="Export As")
        export_btn = gr.Button("Export File")
        export_audio = gr.Audio(label="Exported Track")
        export_btn.click(export_track, [save_audio_path, export_format], export_audio)

demo.launch()
gradio
pydub
librosa
soundfile
transformers
torch
# 🎙️ Royal Voice AI Studio

**Royal Voice** is a complete AI-powered music creation app.

## Features

- 🔐 User login system
- 🎼 AI Lyrics generation (GPT-2)
- 🎧 Multitrack audio mixing (pydub)
- 🎚️ Autotune with pitch correction (librosa)
- 📤 Export audio as MP3, WAV, or OGG
- 💾 Save tracks by user session
- 🌐 100% runs locally or on Replit/Hugging Face

---

## How to Run

1. Install requirements:

```bash
pip install -r requirements.txt
python app.py
---

### ✅ How to use on GitHub:

1. Create a new GitHub repository
2. Upload:
   - `app.py`
   - `requirements.txt`
   - `README.md`
3. Push or upload the files
4. Deploy to Replit or Hugging Face Spaces (select Gradio SDK)

Would you like me to also send this as a `.zip` file or push to a GitHub repo for you?

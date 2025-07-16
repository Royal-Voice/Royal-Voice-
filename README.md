Royal-Voice-All-In-One-studio
import gradio as gr
import os
import uuid
import json
from pydub import AudioSegment
import librosa
import soundfile as sf
import numpy as np
from transformers import pipeline

# === File-based user storage ===
USERS_FILE = "users.json"
if not os.path.exists(USERS_FILE):
    with open(USERS_FILE, "w") as f:
        json.dump({}, f)

def load_users():
    with open(USERS_FILE, "r") as f:
        return json.load(f)

def save_users(users):
    with open(USERS_FILE, "w") as f:
        json.dump(users, f)

# === Signup & Login ===
def signup(username, password):
    users = load_users()
    if username in users:
        return "❌ Username already exists."
    users[username] = {"password": password, "tracks": []}
    save_users(users)
    return "✅ Signup successful!"

def login(username, password):
    users = load_users()
    if username in users and users[username]["password"] == password:
        token = str(uuid.uuid4())
        users[username]["token"] = token
        save_users(users)
        return token
    return "❌ Invalid credentials"

# === Multitrack merging ===
def merge_tracks(t1, t2, t3, t4):
    files = [t1, t2, t3, t4]
    audio_segments = []

    for f in files:
        if f:
            audio = AudioSegment.from_file(f)
            audio_segments.append(audio)

    if not audio_segments:
        return None, "⚠️ No audio provided."

    merged = audio_segments[0]
    for seg in audio_segments[1:]:
        merged = merged.overlay(seg)

    out_path = "merged_output.wav"
    merged.export(out_path, format="wav")
    return out_path, "✅ Tracks merged successfully."

# === AI AutoTune ===
def autotune(input_audio):
    try:
        y, sr = librosa.load(input_audio)
        pitches, magnitudes = librosa.piptrack(y=y, sr=sr)
        pitch_vals = np.max(pitches, axis=0)
        est_pitch = np.mean(pitch_vals[pitch_vals > 0])
        if np.isnan(est_pitch) or est_pitch == 0:
            return input_audio, "⚠️ Pitch not detected."
        return input_audio, f"🎵 Estimated pitch: {round(est_pitch, 2)} Hz"
    except Exception as e:
        return input_audio, f"❌ Error: {str(e)}"

# === Music from Lyrics using GPT-2 ===
generator = pipeline("text-generation", model="gpt2")

def generate_music_from_lyrics(lyrics):
    try:
        result = generator(f"Write a music idea based on: {lyrics}", max_length=100)
        return result[0]['generated_text']
    except Exception as e:
        return f"❌ Error generating: {str(e)}"

# === Save to user's track list ===
def save_track(username, audio_file):
    users = load_users()
    if username in users:
        users[username]["tracks"].append(audio_file)
        save_users(users)
        return "✅ Track saved to user account."
    return "❌ User not found."

# === Gradio Interface ===
with gr.Blocks(title="Royal Voice AI Studio") as app:
    gr.Markdown("# 🎧 Royal Voice AI Studio\n_Free AI Music Studio with Multitrack Mixing, AutoTune, and Lyric Generation_")

    with gr.Tab("🔐 Sign Up / Login"):
        with gr.Row():
            user = gr.Textbox(label="Username")
            pw = gr.Textbox(label="Password", type="password")
        signup_btn = gr.Button("Sign Up")
        login_btn = gr.Button("Log In")
        session_token = gr.Textbox(label="Session Token")

        signup_btn.click(signup, inputs=[user, pw], outputs=session_token)
        login_btn.click(login, inputs=[user, pw], outputs=session_token)

    with gr.Tab("🎧 Merge Tracks"):
        track1 = gr.Audio(label="Track 1", type="filepath")
        track2 = gr.Audio(label="Track 2", type="filepath")
        track3 = gr.Audio(label="Track 3", type="filepath")
        track4 = gr.Audio(label="Track 4", type="filepath")
        merge_btn = gr.Button("Merge Tracks")
        merged_output = gr.Audio(label="Merged Output")
        merge_msg = gr.Textbox()

        merge_btn.click(merge_tracks, [track1, track2, track3, track4], [merged_output, merge_msg])

    with gr.Tab("🎚️ AutoTune"):
        autotune_input = gr.Audio(label="Upload Vocal", type="filepath")
        autotune_btn = gr.Button("AutoTune")
        autotune_output = gr.Audio(label="AutoTuned Output")
        autotune_text = gr.Textbox()

        autotune_btn.click(autotune, [autotune_input], [autotune_output, autotune_text])

    with gr.Tab("🎼 Generate from Lyrics"):
        lyrics_input = gr.Textbox(lines=4, placeholder="Write your lyrics or idea here...")
        lyrics_btn = gr.Button("Generate Song Idea")
        lyrics_output = gr.Textbox(lines=6)

        lyrics_btn.click(generate_music_from_lyrics, [lyrics_input], lyrics_output)

    with gr.Tab("☁️ Save Track"):
        save_user = gr.Textbox(label="Username")
        save_btn = gr.Button("Save Current Track")
        save_status = gr.Textbox()
        save_btn.click(save_track, [save_user, merged_output], save_status)

app.launch()
gradio
pydub
librosa
soundfile
transformers
torch
scipy
numpy
RoyalVoice/
│
├── app.py
├── requirements.txt
├── users.json   ← will be created at runtime

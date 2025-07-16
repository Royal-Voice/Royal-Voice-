Royal-Voice-studio 
import gradio as gr
import uuid
import os
from pydub import AudioSegment
import librosa
import soundfile as sf
import numpy as np

# --- USER SYSTEM ---
users = {"admin": "admin123"}
sessions = {}

def login(username, password):
    if users.get(username) == password:
        token = str(uuid.uuid4())
        sessions[token] = username
        return f"✅ Login successful. Token: {token}"
    return "❌ Invalid credentials"

# --- AI AUTOTUNE ---
def autotune(audio_file):
    try:
        y, sr = librosa.load(audio_file)
        y_tuned = librosa.effects.pitch_shift(y, sr, n_steps=2)
        output_file = "autotuned.wav"
        sf.write(output_file, y_tuned, sr)
        return output_file
    except Exception as e:
        return None

# --- MULTITRACK MIXING ---
def apply_effects(track1, track2, track3, track4, fx, eq_level):
    try:
        combined = None
        tracks = [track1, track2, track3, track4]

        for t in tracks:
            if t is not None:
                audio = AudioSegment.from_file(t)
                if fx == "Reverb":
                    audio = audio + 5
                elif fx == "Echo":
                    audio = audio.overlay(audio, delay=200)
                if eq_level > 50:
                    audio = audio.high_pass_filter(300)
                combined = audio if combined is None else combined.overlay(audio)

        if combined:
            out_path = "mixed_output.wav"
            combined.export(out_path, format="wav")
            return out_path, f"✅ Mixed with {fx}, EQ: {eq_level}%"
        return None, "⚠️ Please upload at least one track."
    except Exception as e:
        return None, "❌ Error during mixing."

# --- EXPORT FILE ---
def export_file(audio_file, fmt):
    try:
        audio = AudioSegment.from_file(audio_file)
        out_path = f"exported.{fmt}"
        audio.export(out_path, format=fmt)
        return out_path, f"✅ Exported as .{fmt}"
    except Exception as e:
        return None, "❌ Export failed."

# --- GENERATE MELODY (MOCK) ---
def generate_music(lyrics):
    return f"🎶 AI-generated melody for: '{lyrics[:100]}...'"


# === GRADIO UI ===
with gr.Blocks(title="Royal Voice Studio") as demo:
    gr.Markdown("# 🎵 Royal Voice Studio 🎤")
    gr.Markdown("Free AI-powered music production tool")

    with gr.Tab("🔐 Login"):
        username = gr.Textbox(label="Username")
        password = gr.Textbox(label="Password", type="password")
        login_btn = gr.Button("Login")
        login_result = gr.Textbox(label="Result")
        login_btn.click(fn=login, inputs=[username, password], outputs=login_result)

    with gr.Tab("🎛️ Multitrack Mixing"):
        gr.Markdown("Upload up to 4 tracks and apply effects")
        track1 = gr.Audio(label="Track 1", type="filepath")
        track2 = gr.Audio(label="Track 2", type="filepath")
        track3 = gr.Audio(label="Track 3", type="filepath")
        track4 = gr.Audio(label="Track 4", type="filepath")
        fx = gr.Dropdown(["None", "Reverb", "Echo"], label="Effect")
        eq = gr.Slider(0, 100, value=50, label="EQ Level")
        mix_btn = gr.Button("Mix Tracks")
        mix_result = gr.Audio(label="Mixed Output")
        mix_msg = gr.Textbox(label="Message")
        mix_btn.click(fn=apply_effects, inputs=[track1, track2, track3, track4, fx, eq], outputs=[mix_result, mix_msg])

    with gr.Tab("🎤 AI AutoTune"):
        vocal = gr.Audio(label="Upload Raw Vocal", type="filepath")
        tune_btn = gr.Button("Apply AutoTune")
        tuned_result = gr.Audio(label="Tuned Output")
        tune_btn.click(fn=autotune, inputs=vocal, outputs=tuned_result)

    with gr.Tab("🎶 Generate Music from Lyrics"):
        lyrics = gr.Textbox(label="Lyrics")
        gen_btn = gr.Button("Generate")
        gen_result = gr.Textbox(label="AI Melody")
        gen_btn.click(fn=generate_music, inputs=lyrics, outputs=gen_result)

    with gr.Tab("📤 Export"):
        export_fmt = gr.Dropdown(["mp3", "wav", "ogg"], label="Format")
        export_btn = gr.Button("Export Final Audio")
        export_result = gr.Audio(label="Exported Audio")
        export_msg = gr.Textbox(label="Status")
        export_btn.click(fn=export_file, inputs=[mix_result, export_fmt], outputs=[export_result, export_msg])

    with gr.Tab("📱 PWA Install"):
        gr.Markdown("To install this app on mobile:\n- Tap the browser menu\n- Choose 'Add to Home Screen'")

demo.launch()
gradio
pydub
librosa
soundfile
numpy
scipy
python app.py
app.py
requirements.txt

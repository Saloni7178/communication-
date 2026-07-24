# communication
Creators upaay
/n
Author = creators



"""
AI Speaking Coach — Frontend (Streamlit)
------------------------------------------
This is the FRONTEND layer only, matching the flow:
  1. User Interacts (Frontend)
     - Streamlit renders the interface
     - User clicks "Record" and speaks a practice response into the mic
     - The recorded audio is captured and held in session state,
       ready to be passed downstream (e.g. to a speech-to-text +
       LLM feedback pipeline in a later step).

Requires:
    pip install streamlit audio-recorder-streamlit

Run with:
    streamlit run app.py
"""

import streamlit as st
from audio_recorder_streamlit import audio_recorder
from datetime import datetime

# ---------------------------------------------------------------------------
# Page config
# ---------------------------------------------------------------------------
st.set_page_config(
    page_title="AI Speaking Coach",
    page_icon="🎙️",
    layout="centered",
)

# ---------------------------------------------------------------------------
# Session state init
# ---------------------------------------------------------------------------
if "recordings" not in st.session_state:
    st.session_state.recordings = []  # list of dicts: {audio_bytes, timestamp}

if "latest_audio" not in st.session_state:
    st.session_state.latest_audio = None

# ---------------------------------------------------------------------------
# Header
# ---------------------------------------------------------------------------
st.title("🎙️ AI Speaking Coach")
st.markdown(
    "Practice speaking with confidence and clarity. "
    "Choose a mode below, hit **Record**, and speak your response."
)

# ---------------------------------------------------------------------------
# Practice mode selector
# ---------------------------------------------------------------------------
practice_mode = st.selectbox(
    "Choose your practice mode",
    [
        "General Voice Conversation Practice",
        "Interview Practice",
        "Presentation / Public Speaking Practice",
        "Articulation & Fluency Drill",
    ],
)

prompt_bank = {
    "General Voice Conversation Practice": "Tell me about a recent challenge you overcame.",
    "Interview Practice": "Tell me about yourself and why you're a good fit for this role.",
    "Presentation / Public Speaking Practice": "Give a 60-second pitch introducing your project.",
    "Articulation & Fluency Drill": "Read this sentence clearly: 'She sells seashells by the seashore.'",
}

st.info(f"**Prompt:** {prompt_bank[practice_mode]}")

st.divider()

# ---------------------------------------------------------------------------
# Recording section
# ---------------------------------------------------------------------------
st.subheader("Record your response")

col1, col2 = st.columns([1, 3])

with col1:
    audio_bytes = audio_recorder(
        text="",
        recording_color="#6c5ce7",
        neutral_color="#a29bfe",
        icon_name="microphone",
        icon_size="3x",
        pause_threshold=2.0,
    )

with col2:
    st.caption("Click the mic to start recording. Click again to stop.")

# ---------------------------------------------------------------------------
# Handle new recording
# ---------------------------------------------------------------------------
if audio_bytes:
    st.session_state.latest_audio = audio_bytes
    st.session_state.recordings.append(
        {
            "audio_bytes": audio_bytes,
            "mode": practice_mode,
            "prompt": prompt_bank[practice_mode],
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        }
    )

    st.success("Recording captured!")
    st.audio(audio_bytes, format="audio/wav")

    # Placeholder call-out for the next pipeline stage
    # (speech-to-text -> LLM analysis -> feedback rendering)
    if st.button("Get Feedback", type="primary"):
        with st.spinner("Analyzing your speech..."):
            # TODO: send `audio_bytes` to backend pipeline:
            #   1. Speech-to-text transcription
            #   2. LLM-based analysis (clarity, confidence, articulation)
            #   3. Return structured feedback + progress tracking update
            st.info(
                "Backend analysis not yet connected. This is where "
                "transcription + AI feedback will appear."
            )

# ---------------------------------------------------------------------------
# Session history / progress tracking (placeholder UI)
# ---------------------------------------------------------------------------
if st.session_state.recordings:
    st.divider()
    st.subheader("Your Session History")
    for i, rec in enumerate(reversed(st.session_state.recordings), start=1):
        with st.expander(f"Recording {len(st.session_state.recordings) - i + 1} — {rec['timestamp']}"):
            st.write(f"**Mode:** {rec['mode']}")
            st.write(f"**Prompt:** {rec['prompt']}")
            st.audio(rec["audio_bytes"], format="audio/wav")

# ---------------------------------------------------------------------------
# Sidebar — progress tracking placeholder
# ---------------------------------------------------------------------------
with st.sidebar:
    st.header("📈 Progress")
    st.metric("Sessions completed", len(st.session_state.recordings))
    st.caption("Confidence, clarity, and fluency trends will appear here once feedback analysis is connected.")
    # Insert the api key here
genai.configure(api_key="")

SYSTEM_PROMPT = """
You are an expert communication coach. You will be given the transcript 
of a user's spoken response. Analyze it for:
- Clarity of message
- Filler words (um, uh, like, etc.)
- Pacing and structure
- Confidence and tone

Then provide:
1. A score out of 10 for overall communication effectiveness
2. 3 specific, actionable pieces of feedback
3. One rewritten example sentence showing improvement

Respond in a clear, structured format.
"""

model = genai.GenerativeModel(
    model_name="gemini-1.5-pro",
    system_instruction=SYSTEM_PROMPT
)


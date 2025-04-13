# resume-reviewer
import streamlit as st
from PyPDF2 import PdfReader
from transformers import pipeline
from fpdf import FPDF
import base64

# ---------- PAGE CONFIG ----------
st.set_page_config(page_title="AI Resume Reviewer", page_icon="🤖", layout="centered")
st.title("⚔ AI Resume Reviewer – Hackathon Edition")
st.markdown("Welcome! Upload your resume and get AI-powered feedback based on your target role or job description.")
st.divider()


# ---------- SIDEBAR ----------
with st.sidebar:
    st.title("🛠 Settings")
    dark_mode = st.toggle("🌙 Enable Dark Mode")  # DARK MODE TOGGLE
    st.markdown("(For demo purposes, no API key required)")
    st.text_input("🔑 Hugging Face API Token", type="password", disabled=True)
    st.markdown("👥 *Team:* Dev, Sumit, Pawan, Advitiya")

# ---------- DARK MODE STYLES ----------
if dark_mode:
    st.markdown(
        """
        <style>
        .stApp {
            background-color: #0e1117;
            color: #f5f5f5;
        }
        div[data-testid="stSidebar"] {
            background-color: #1c1f26;
        }
        h1, h2, h3, h4, h5, h6, p, label, span, .stMarkdown {
            color: #f5f5f5 !important;
        }
        .stButton>button {
            background-color: #252930;
            color: white;
            border-radius: 10px;
        }
        </style>
        """,
        unsafe_allow_html=True
    )

# ---------- UTILS ----------
def generate_pdf(feedback_text, filename="feedback.pdf"):
    pdf = FPDF()
    pdf.add_page()
    pdf.set_font("Arial", size=12)
    for line in feedback_text.split('\n'):
        try:
            pdf.multi_cell(0, 10, line.encode('latin-1', 'replace').decode('latin-1'))
        except:
            pdf.multi_cell(0, 10, line)
    pdf.output(filename)
    return filename

def get_pdf_download_link(pdf_file_path):
    with open(pdf_file_path, "rb") as f:
        base64_pdf = base64.b64encode(f.read()).decode('utf-8')
    href = f'<a href="data:application/octet-stream;base64,{base64_pdf}" download="{pdf_file_path}">📄 Download Feedback as PDF</a>'
    return href

# ---------- TABS ----------
tab1, tab2, tab3 = st.tabs(["📄 Upload & Analyze", "⚙ Rule-Based Review", "📊 JD Comparison"])

resume_text = ""
job_role = ""

# ---------- Hugging Face Model ----------
generator = pipeline("text-generation", model="gpt2")

# ---------- TAB 1 ----------
with tab1:
    st.header("📄 Resume Upload & Hugging Face Analysis")
    uploaded_file = st.file_uploader("Upload your Resume (PDF)", type=["pdf"])
    job_role = st.selectbox("🎯 Choose your Target Role", [
        "Web Developer", "Data Analyst", "UI/UX Designer", 
        "ML Engineer", "Mobile App Developer", "Cybersecurity Analyst"
    ])

    if uploaded_file is not None:
        reader = PdfReader(uploaded_file)
        resume_text = "\n".join([page.extract_text() for page in reader.pages if page.extract_text()])

    if st.button("🧠 Analyze with Hugging Face (Demo)"):
        if resume_text:
            try:
                prompt = f"Analyze this resume for the role of {job_role}: {resume_text[:1000]}"
                feedback = generator(prompt, max_length=300)
                st.subheader("📌 Hugging Face Feedback:")
                st.success(feedback[0]['generated_text'])

                generate_pdf("Hugging Face Feedback:\n" + feedback[0]['generated_text'], "hugging_face_feedback.pdf")
                st.markdown(get_pdf_download_link("hugging_face_feedback.pdf"), unsafe_allow_html=True)
            except Exception as e:
                st.error(f"Error generating feedback: {e}")
        else:
            st.warning("Please upload a resume first.")

# ---------- TAB 2 ----------
with tab2:
    st.header("🧪 Rule-Based Resume Review")
    if resume_text:
        if st.button("⚙ Run Rule-Based Feedback"):
            rule_feedback = ""
            if "Objective" not in resume_text:
                rule_feedback += "- ❌ Missing Objective section.\n"
            if "Education" not in resume_text:
                rule_feedback += "- ❌ Missing Education section.\n"
            if "Project" not in resume_text:
                rule_feedback += "- ❌ Missing Projects.\n"
            if "Internship" not in resume_text:
                rule_feedback += "- ❌ Missing Internship experience.\n"
            if rule_feedback == "":
                rule_feedback = "✅ Resume looks complete!"
            st.subheader("📌 Rule-Based Feedback:")
            st.info(rule_feedback)
    else:
        st.warning("Please upload a resume in Tab 1 first.")

# ---------- TAB 3 ----------
with tab3:
    st.header("📋 Compare Resume with Job Description")
    job_description = st.text_area("Paste the Job Description below", height=200)

    if st.button("📊 Compare Resume with JD (Demo)"):
        if resume_text and job_description:
            jd_feedback = (
                "✅ Skills matched: Python, SQL, Data Analysis\n"
                "❌ Missing: Cloud tools (AWS), Business communication\n\n"
                "*Suggestions:*\n"
                "- Add keywords from the JD.\n"
                "- Align past experience with job responsibilities.\n"
                "- Emphasize relevant projects or tools used."
            )
            st.subheader("📌 JD Match Feedback:")
            st.success(jd_feedback)

            generate_pdf("JD Match Feedback:\n" + jd_feedback, "jd_feedback.pdf")
            st.markdown(get_pdf_download_link("jd_feedback.pdf"), unsafe_allow_html=True)
        else:
            st.warning("Please upload resume and enter job description.")

# ---------- HIDE FOOTER ----------
st.markdown("""
    <style>
    #MainMenu {visibility: hidden;}
    footer {visibility: hidden;}
    </style>
""", unsafe_allow_html=True)

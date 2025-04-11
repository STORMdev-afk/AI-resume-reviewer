import streamlit as st
from PyPDF2 import PdfReader
from fpdf import FPDF
import base64
import re

# ---------- PAGE CONFIG ----------
st.set_page_config(page_title="AI Resume Reviewer", page_icon="🤖", layout="centered")
st.title("AI Resume Reviewer – Hackathon Edition")
st.markdown("Upload your resume and get AI-powered feedback based on your target role or job description.")
st.divider()

# ---------- SIDEBAR ----------
with st.sidebar:
    st.title("🛠️ Settings")
    st.markdown("*(For demo purposes, API key not required)*")
    st.text_input("🔑 OpenAI API Key", type="password", disabled=True)
    st.markdown("👥 **Team:** Dev, Sumit, Pawan, Advitya")

# ---------- SESSION STATE ----------
if "resume_text" not in st.session_state:
    st.session_state.resume_text = ""

# ---------- UTILS ----------
def remove_non_latin(text):
    return re.sub(r'[^\x00-\x7F]+', '', text)

def generate_pdf(feedback_text, filename="feedback.pdf"):
    pdf = FPDF()
    pdf.add_page()
    pdf.set_font("Arial", size=12)
    cleaned_text = remove_non_latin(feedback_text)
    for line in cleaned_text.split('\n'):
        pdf.multi_cell(0, 10, line)
    pdf.output(filename)
    return filename

def get_pdf_download_link(pdf_file_path):
    with open(pdf_file_path, "rb") as f:
        base64_pdf = base64.b64encode(f.read()).decode('utf-8')
    href = f'<a href="data:application/octet-stream;base64,{base64_pdf}" download="{pdf_file_path}">📄 Download Feedback as PDF</a>'
    return href

# ---------- TABS ----------
tab1, tab2, tab3 = st.tabs(["📄 Upload & Analyze", "⚙️ Rule-Based Review", "📊 JD Comparison"])

# ---------- TAB 1 ----------
with tab1:
    st.header("📄 Resume Upload & GPT Analysis (Demo)")
    uploaded_file = st.file_uploader("Upload your Resume (PDF)", type=["pdf"])
    job_role = st.selectbox("🎯 Choose your Target Role", [
        "Web Developer", "Data Analyst", "UI/UX Designer", 
        "ML Engineer", "Mobile App Developer", "Cybersecurity Analyst"
    ])

    if uploaded_file is not None:
        reader = PdfReader(uploaded_file)
        st.session_state.resume_text = "\n".join([page.extract_text() for page in reader.pages if page.extract_text()])

    if st.button("Analyze with GPT (Demo)"):
        if st.session_state.resume_text:
            gpt_feedback = (
                f"Resume Feedback for {job_role}:\n\n"
                "- Add a clear Objective tailored to the role.\n"
                "- Highlight relevant skills (e.g., JavaScript, SQL).\n"
                "- Showcase personal projects and GitHub links.\n"
                "- Improve formatting and include measurable achievements.\n"
                "- Mention any certifications or online courses."
            )
            st.subheader("📌 GPT Feedback:")
            st.success(gpt_feedback)

            generate_pdf("GPT Feedback:\n" + gpt_feedback, "gpt_feedback.pdf")
            st.markdown(get_pdf_download_link("gpt_feedback.pdf"), unsafe_allow_html=True)
        else:
            st.warning("Please upload a resume first.")

# ---------- TAB 2 ----------
with tab2:
    st.header("🧪 Rule-Based Resume Review")
    if st.session_state.resume_text:
        if st.button("Run Rule-Based Feedback"):
            resume_text = st.session_state.resume_text
            rule_feedback = ""
            if "Objective" not in resume_text:
                rule_feedback += "- Missing Objective section.\n"
            if "Education" not in resume_text:
                rule_feedback += "- Missing Education section.\n"
            if "Project" not in resume_text:
                rule_feedback += "- Missing Projects.\n"
            if "Internship" not in resume_text:
                rule_feedback += "- Missing Internship experience.\n"
            if rule_feedback == "":
                rule_feedback = "Resume looks complete!"
            st.subheader("📌 Rule-Based Feedback:")
            st.info(rule_feedback)
    else:
        st.warning("Please upload a resume in Tab 1 first.")

# ---------- TAB 3 ----------
with tab3:
    st.header("📋 Compare Resume with Job Description")
    job_description = st.text_area("Paste the Job Description below", height=200)

    if st.button("Compare Resume with JD (Demo)"):
        if st.session_state.resume_text and job_description:
            jd_feedback = (
                "Skills matched: Python, SQL, Data Analysis\n"
                "Missing: Cloud tools (AWS), Business communication\n\n"
                "Suggestions:\n"
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
hide_streamlit_style = """
    <style>
    #MainMenu {visibility: hidden;}
    footer {visibility: hidden;}
    </style>
"""
st.markdown(hide_streamlit_style, unsafe_allow_html=True)

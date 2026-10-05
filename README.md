import streamlit as st
import openai

# Set page configuration
st.set_page_config(page_title="AI Comic Story Generator", layout="wide")

st.title("📖 AI Comic Story Generator")
st.write("Generate unique comic stories and panel artwork using AI!")

# Sidebar for API Key input
api_key = st.sidebar.text_input("Enter OpenAI API Key", type="password")

# User inputs
prompt = st.text_area("Describe your comic plot or idea:", "A brave space cat exploring a synthwave neon planet.")
num_panels = st.slider("Number of Panels", min_value=2, max_value=6, value=4)

if st.button("Generate Comic") and api_key:
    client = openai.OpenAI(api_key=api_key)
    
    with st.spinner("Writing script and panel prompts..."):
        # Step 1: Generate Story Script and Image Prompts
        script_prompt = f"""
        Create a {num_panels}-panel comic script based on this idea: '{prompt}'.
        For each panel, provide:
        1. Panel Description (Visual description of the scene)
        2. Caption/Dialogue (Text inside the panel)
        Format as clear numbered panels.
        """
        
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": script_prompt}]
        )
        script = response.choices[0].message.content
        st.subheader("📜 Generated Script")
        st.write(script)

    with st.spinner("Generating panel artwork..."):
        # Step 2: Generate Image for Panel 1 (Example)
        st.subheader("🎨 Panel Artwork")
        cols = st.columns(num_panels)
        
        for i in range(num_panels):
            image_prompt = f"Comic book style illustration, panel {i+1} of a story about {prompt}, high detail, vibrant colors."
            img_response = client.images.generate(
                model="dall-e-3",
                prompt=image_prompt,
                n=1,
                size="1024x1024"
            )
            image_url = img_response.data[0].url
            
            with cols[i]:
                st.image(image_url, caption=f"Panel {i+1}")

elif not api_key:
    st.info("Please enter your OpenAI API key in the sidebar to generate the comic.")

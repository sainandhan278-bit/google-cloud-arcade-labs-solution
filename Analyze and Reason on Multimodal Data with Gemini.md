# Analyze and Reason on Multimodal Data with Gemini: Challenge Lab

Python and Gemini API solutions for the Analyze and Reason on Multimodal Data with Gemini Challenge Lab using the `gemini-3.5-flash` model.

> **Note**: if you encounter an authentication error when running the cells in the notebook, go to **Agent Platform > Dashboard**, and click on **Enable All Recommended APIs**. Then, re-run the failed cell, and continue the lab.

> Run cells top to bottom in order. Always edit the existing TODO cells — do NOT add new cells. Task 5 must run the gcloud upload cell to trigger the grade check.

## Author

**Aayush Shrestha (Shinux)** — Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

---

## Task 1. Open the notebook in Agent Platform Workbench

> Just run all pre-written cells (No TODOs). Press `Shift+Enter` on each cell. Nothing to fill.

## Task 2. Analyze and reason on text data

### Initial Analysis with Gemini Flash

```python
# 2. Construct the prompt for Gemini
prompt = f"""
Please analyze these customer reviews and social media posts regarding Cymbal Direct's new athletic wear.
For every single post or review, you must:
1. Determine if the sentiment is positive, negative, or neutral.
2. Pull out the main topics discussed, such as pricing, fit, style, quality of the product, and customer service.
3. Point out any specific features or product names that are mentioned frequently.

Here is the data:
{text_data}
"""

# 3. Send the prompt to Gemini
response = client.models.generate_content(
    model=MODEL_ID,
    contents=prompt
)

# 4. Display the response
display(Markdown(response.text))
```

### Deep dive with Gemini Flash: Reasoning on customer sentiment

```python
# 1. Construct the prompt for Gemini
thinking_mode_prompt = f"""
Take a deeper look at the customer feedback and provide a detailed analysis:
- What are the core reasons driving both the negative and positive sentiments?
- How is the new athletic apparel line affecting Cymbal Direct's overall brand perception?
- Give me exactly three main areas where Cymbal Direct needs to improve its products or customer satisfaction.
- Finally, assuming you are presenting this to the marketing team, what are the three most critical takeaways?

Feedback data:
{text_data}
"""

# 2. Send the prompt to the Gemini Thinking model
thinking_model_response = client.models.generate_content(
    model=MODEL_ID,
    contents=thinking_mode_prompt,
    config=config
)

# 3. Print thoughts and answer
print_thoughts(thinking_model_response)

# 4. Save the text analysis to a file
with open('analysis/text_analysis.md', 'w') as f:
    f.write(thinking_model_response.text)
```

## Task 3. Analyze and reason on visual content

### Initial Analysis with Gemini Flash

```python
# 3. Construct the prompt for Gemini
prompt = """
Review these images featuring Cymbal Direct's new line of athletic apparel and do the following:
- List the specific apparel items visible in each photo (like shoes, shirts, accessories, leggings).
- Detail the characteristics of the items, including their style, fit, design, and color.
- Highlight any obvious customer preferences or overarching style trends seen across the pictures.
"""

# 4. Send the prompt and images to Gemini
response = client.models.generate_content(
    model=MODEL_ID,
    contents=[prompt] + image_parts
)

# 5. Display the response
display(Markdown(response.text))
```

### Reasoning on image trends with Gemini Flash

```python
# 1. Construct the prompt for Gemini
thinking_mode_prompt = """
I need a deeper analysis of these Cymbal Direct apparel images:
- For each image, what is your hypothesis regarding the target demographic (lifestyle, age, gender, fitness level)?
- How do the visual components (models, backgrounds, poses, colors) add to the overall appeal and brand message?
- Compare the trends you see here against the wider trends in the athletic fashion industry.
- Give concrete recommendations for product development or future marketing efforts for Cymbal Direct.
"""

# 2. Send the prompt and images to the Gemini Thinking model
thinking_model_response_image = client.models.generate_content(
    model=MODEL_ID,
    contents=[thinking_mode_prompt] + image_parts,
    config=config
)

# 3. Print thoughts and answer
print_thoughts(thinking_model_response_image)

# 4. Save the image analysis to a file
with open('analysis/image_analysis.md', 'w') as f:
    f.write(thinking_model_response_image.text)
```

## Task 4. Analyze and reason on audio content

### Initial analysis with Gemini Flash

```python
# 1. Construct the prompt for Gemini
prompt = """
Listen to this podcast audio covering Cymbal Direct's new apparel line and:
1. Provide a transcription of the chat, making sure to clearly separate different speakers.
2. Perform a sentiment analysis that points out neutral, negative, and positive opinions.
3. Extract the primary themes of the discussion (e.g., style, performance, fit, comfort, competitor comparisons).
4. Give a summary of the overall perception regarding the new clothing line.
"""

# 2. Send the prompt and audio to Gemini
response = client.models.generate_content(
    model=MODEL_ID,
    contents=[prompt, audio_part]
)

# 3. Display the response
display(Markdown(response.text))
```

### Reasoning on audio insights with Gemini Flash

```python
# 1. Construct the prompt for Gemini
thinking_mode_prompt = """
Provide an in-depth analysis of this podcast interview about Cymbal Direct:
- Determine the overall customer satisfaction levels with the new apparel based on the conversation.
- Figure out what key factors are driving how customers perceive the brand.
- Create exactly three data-driven, actionable recommendations for Cymbal Direct.
- Point out any limitations or potential biases present in this audio recording.
"""

# 2. Use Gemini thinking for deeper reasoning
thinking_model_response = client.models.generate_content(
    model=MODEL_ID,
    contents=[thinking_mode_prompt, audio_part],
    config=config
)

# 3. Print thoughts and answer
print_thoughts(thinking_model_response)

# 4. Save the audio analysis to a file
with open('analysis/audio_analysis.md', 'w') as f:
    f.write(thinking_model_response.text)
```

## Task 5. Synthesize multimodal insights

### Generate a comprehensive report

```python
# 1. Load the analysis results from the files
with open('analysis/text_analysis.md', 'r') as f:
    text_analysis = f.read()

with open('analysis/image_analysis.md', 'r') as f:
    image_analysis = f.read()

with open('analysis/audio_analysis.md', 'r') as f:
    audio_analysis = f.read()

# 2. Combine the analysis results
all_analysis = f"""
## Text Analysis:
{text_analysis}

## Image Analysis:
{image_analysis}

## Audio Analysis:
{audio_analysis}
"""

# 3. Construct the prompt for Gemini
comprehensive_report_prompt = f"""
Assume the role of a senior marketing analyst for Cymbal Direct. I am providing you with analysis results from audio, image, and text data concerning our new athletic apparel line. Write a comprehensive report that:
- Gives a summary of the overall sentiment surrounding the new line.
- Highlights the major trends and themes found in the customer feedback.
- Delivers insights regarding customer behavior, usage patterns, and style preferences.
- Discusses the audio feedback and how well it aligns with the product images.
- Proposes actionable steps and recommendations for Cymbal Direct to improve product positioning and marketing strategy.

Please format your response as a professional Markdown report, complete with appropriate sections and headings.

Here is the combined analysis:
{all_analysis}
"""

# 4. Send the prompt to Gemini
thinking_model_response = client.models.generate_content(
    model=MODEL_ID,
    contents=comprehensive_report_prompt,
    config=config
)

# 5. Print the thoughts and answer
print_thoughts(thinking_model_response)

# 6. Save the final report to a file
with open('analysis/final_report.md', 'w') as f:
    f.write(thinking_model_response.text)
```

### Upload to Cloud Storage

```python
!gcloud storage cp analysis/final_report.md gs://{PROJECT_ID}-bucket/analysis/final_report.md
```

---

## Author

**Aayush Shrestha (Shinux)** — Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

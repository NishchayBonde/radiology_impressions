# Radiology Impression Generator with Document Loader

## 🎯 Overview

A streamlit application that generates radiology impressions by passing **complete reference documents** to GPT-4, ensuring ALL guidelines and protocols are followed. Features study-specific prompts fully editable from the UI.

## ✨ Key Features

### 1. **Complete Document Loading**
- Loads entire DOCX files (protocols, guidelines, checklists)
- Preserves document structure (headings, paragraphs, tables)
- No chunking or retrieval - AI sees everything
- Better for ensuring all steps are followed

### 2. **Dynamic Prompt Editing from UI**
- Edit prompts directly in the frontend
- Changes saved to backend (`study_prompts.json`)
- Use `{document_content}` placeholder for document insertion
- Preview token usage before saving

### 3. **Study-Specific Configuration**
- Each study type has its own:
  - Custom prompt template
  - Reference document (DOCX)
- Managed entirely from the UI - no code changes needed

### 4. **Smart Document Loading**
- Uses `docx2txt` for better text extraction
- Falls back to `python-docx` with structure preservation
- Extracts tables and formats them readably
- Identifies headers and creates visual hierarchy

### 5. **YAML-Based Study Management**
- Study types loaded from `studies.yaml`
- Add/remove studies by editing the YAML file
- Changes reflected immediately (with cache refresh)

## 📦 Installation

```bash
# Clone or download the files
pip install -r requirements.txt

# Run the app
streamlit run radiology_impressions.py
```

## 🚀 Quick Start

### 1. Configure a Study Type

1. Open the app
2. Enter your OpenAI API key in sidebar
3. Enable "Configuration Mode" in sidebar
4. Select a study type (e.g., "CT Chest")
5. Click "Manage Prompt Template":
   ```
   You are an expert radiologist for {STUDY_TYPE}.
   
   CRITICAL: Follow ALL steps in the reference document below.
   
   {document_content}
   
   Generate 3-4 bullet points following the document exactly.
   ```
6. Save the prompt
7. Click "Manage Reference Document"
8. Upload your protocol DOCX file
9. Done!

### 2. Generate an Impression

1. Select your study type
2. Enter clinical history (optional)
3. Paste radiologist findings
4. Click "Generate Impression"
5. Edit the result if needed
6. Download or copy

## 📁 File Structure

```
your_project/
├── app_doc.py                          # Main application
├── requirements.txt                    # Python dependencies
├── config/                             # Configuration directory
│   ├── studies.yaml                   # Study types list
│   ├── study_prompts.json            # Custom prompts
│   └── study_documents/              # Reference documents
│       ├── CT_Chest.docx
│       ├── MRI_Brain.docx
│       └── ...
```

## 🔧 Configuration Files

### studies.yaml
```yaml
studies:
  - CT Chest
  - CT Abdomen/Pelvis
  - MRI Brain
  # Add more studies here
```

### study_prompts.json (auto-generated)
```json
{
  "CT Chest": "Your custom prompt with {document_content} placeholder",
  "MRI Brain": "Another custom prompt..."
}
```

## 💡 Best Practices

### For Reference Documents

✅ **Good Documents:**
- Step-by-step protocols
- Checklists and templates
- Measurement standards
- Reporting guidelines
- Classification systems (CAD-RADS, BI-RADS, etc.)

📏 **Size Considerations:**
- Optimal: 10-50 pages (~20-100k tokens)
- Maximum: ~80 pages (120k token limit with gpt-4o)
- If larger: Consider splitting into multiple focused documents

### For Prompts

**Use the `{document_content}` placeholder:**
```python
"""You are an expert radiologist.

INSTRUCTIONS:
1. Read the COMPLETE reference document below
2. Follow EVERY step and protocol mentioned
3. Apply ALL relevant guidelines and standards

---REFERENCE DOCUMENT---
{document_content}
---END DOCUMENT---

Generate impression following the document exactly."""
```

**Emphasize following the document:**
- "MUST follow ALL steps"
- "Read ENTIRE document"
- "Apply EVERY relevant guideline"
- "Follow document protocols exactly"

### Example Prompt Templates

**For Protocol-Heavy Studies:**
```python
"""You are a {SPECIALTY} radiologist following institutional protocols.

CRITICAL REQUIREMENTS:
1. Read the COMPLETE protocol document below
2. Follow the specified reporting structure exactly
3. Include all required measurements and assessments
4. Use the exact terminology from the document

{document_content}

Clinical Information:
- Study Type: {study_type}
- History: {history}
- Findings: {findings}

Generate a structured impression following the protocol above."""
```

**For Guideline-Based Studies:**
```python
"""Expert radiologist applying society guidelines.

The reference document contains complete guidelines. Your task:
1. Review ALL guidelines in the document
2. Identify relevant criteria for this case
3. Apply appropriate classifications
4. Follow reporting standards

{document_content}

Generate impression per guidelines (3-4 bullet points)."""
```

## 📊 Token Management

### Understanding Context Limits

| Model | Context Window | Recommended Doc Size |
|-------|---------------|---------------------|
| gpt-4o | 128k tokens | Up to 80 pages |
| gpt-4o-mini | 128k tokens | Up to 80 pages |
| gpt-4-turbo | 128k tokens | Up to 80 pages |

### Token Estimation

The app estimates tokens as: `word_count × 1.3`

**Example:**
- 30-page document = ~15,000 words = ~20,000 tokens
- Prompt overhead = ~5,000 tokens
- User input = ~1,000 tokens
- Total = ~26,000 tokens ✅ Well within limits

### What if Document is Too Large?

**Option 1: Summarize the document first**
```python
# Use GPT to create a condensed version
"Summarize this 200-page protocol into key guidelines (10 pages max)"
```

**Option 2: Split into focused documents**
- Instead of: "Complete_CT_Protocols.docx" (200 pages)
- Use: "CT_Chest_Protocol.docx" (30 pages)
- And: "CT_Contrast_Guidelines.docx" (20 pages)

**Option 3: Extract relevant sections manually**
- Keep only the sections needed for reporting
- Remove administrative content, references, etc.

## 🎓 Use Cases & Examples

### Use Case 1: Lung Nodule Reporting

**Document:** Fleischner Society Guidelines (15 pages)
**Content:**
- Size thresholds
- Risk stratification
- Follow-up intervals
- Reporting templates

**Prompt Focus:**
```
"Follow Fleischner criteria for nodule management.
Apply size-based recommendations exactly as specified."
```

**Result:** Consistent, guideline-compliant recommendations

### Use Case 2: Cardiac CT (CAD-RADS)

**Document:** CAD-RADS Classification System (25 pages)
**Content:**
- Stenosis grading
- Plaque characterization
- Modifier assignments
- Reporting format

**Prompt Focus:**
```
"Apply CAD-RADS classification system.
Assign appropriate category and modifiers per document."
```

**Result:** Standardized categorical reporting

### Use Case 3: Institutional Stroke Protocol

**Document:** Hospital Acute Stroke Imaging Protocol (40 pages)
**Content:**
- ASPECTS scoring
- Perfusion criteria
- Treatment windows
- Critical findings workflow

**Prompt Focus:**
```
"Follow institutional stroke protocol exactly.
Include all required elements for acute stroke impression."
```

**Result:** Protocol-compliant, time-critical reporting

## 🔍 How It Works

### Document Loading Pipeline

1. **File Upload** → Saved to `config/study_documents/`
2. **Text Extraction** → `docx2txt` extracts all text
3. **Structure Preservation** → Headers, paragraphs, tables identified
4. **Prompt Assembly** → Document inserted into `{document_content}` placeholder
5. **API Call** → Complete prompt sent to GPT-4
6. **Generation** → AI follows all document guidelines

### Why This Approach?

**Advantages over RAG:**
- ✅ AI sees complete context - nothing missed
- ✅ Can follow multi-step protocols end-to-end
- ✅ No retrieval errors or missing sections
- ✅ Better for procedural/checklist documents
- ✅ Simpler implementation - no vector stores

**Considerations:**
- ⚠️ Requires larger context windows
- ⚠️ Higher token usage per request
- ⚠️ Document size limits (~80 pages max)

**When to use RAG instead:**
- Very large reference libraries (>100 pages)
- Need to search across multiple documents
- Cost optimization is critical
- Documents are reference material, not protocols

## 🛠️ Troubleshooting

### "Context length exceeded" error

**Solution 1:** Use gpt-4o (128k context)
```python
model = "gpt-4o"  # Largest context window
```

**Solution 2:** Reduce document size
- Remove unnecessary sections
- Extract only reporting guidelines
- Summarize verbose content

**Solution 3:** Check token estimation
- Enable Configuration Mode
- View document size estimate
- Aim for <100k tokens total

### Document structure not preserved

**Check extraction method:**
```python
# Install docx2txt for better extraction
pip install docx2txt

# Falls back to python-docx automatically
```

**Verify document format:**
- Use .docx format (not .doc)
- Avoid complex formatting
- Tables should be simple

### AI not following document guidelines

**Strengthen prompt instructions:**
```python
"""CRITICAL INSTRUCTION: You MUST follow EVERY step in the document.

Before generating:
1. Read the ENTIRE reference document
2. Identify ALL applicable protocols
3. Create a mental checklist of requirements

{document_content}

Now apply ALL guidelines above."""
```

**Verify document content:**
- Check document loaded correctly (Preview tab)
- Ensure guidelines are clear and explicit
- Consider adding examples to the document

### Prompt not saving

**Check for placeholder:**
```python
# Correct
"{document_content}"

# Incorrect  
"${document_content}"
"{{document_content}}"
```

**Verify file permissions:**
```bash
chmod -R 755 config/
```

## 📈 Performance & Costs

### Token Usage

**Typical Request:**
- Document: 20,000 tokens
- Prompt: 500 tokens
- User input: 1,000 tokens
- Response: 300 tokens
- **Total: ~21,800 tokens**

### Cost Estimation (gpt-4o-mini)

- Input: $0.150 / 1M tokens
- Output: $0.600 / 1M tokens

**Per impression:**
- Input cost: ~21,500 × $0.15 / 1M = $0.003
- Output cost: 300 × $0.60 / 1M = $0.0002
- **Total: ~$0.0032 per impression**

### Optimization Tips

1. **Reuse documents** - One document serves many impressions
2. **Concise findings** - Keep user input focused
3. **Limit max_tokens** - Set appropriate response length
4. **Use mini model** - gpt-4o-mini is 60% cheaper than gpt-4o

## 🔐 Security & Privacy

- **Local storage:** All documents stored on your machine
- **API calls only:** Only findings + document sent to OpenAI
- **No data retention:** OpenAI doesn't train on API data (by default)
- **Your API key:** You control access and usage

## 🎨 Customization

### Adding New Study Types

**Edit `config/studies.yaml`:**
```yaml
studies:
  - CT Chest
  - CT Cardiac        # New study type
  - PET/CT Oncology   # Another new type
```

Refresh the app - new studies appear immediately!

### Custom Prompt Variables

You can add custom placeholders:

```python
# In prompt template
"""Study: {study_type}
History: {history}
Findings: {findings}

{document_content}"""

# Variables are auto-filled during generation
```

### Styling

The app uses Streamlit's theming. Customize in `.streamlit/config.toml`:

```toml
[theme]
primaryColor = "#1f77b4"
backgroundColor = "#ffffff"
secondaryBackgroundColor = "#f0f2f6"
```

## 📝 License & Credits

**Built with:**
- Streamlit - Web interface
- OpenAI GPT-4 - Impression generation
- python-docx - Document parsing
- docx2txt - Text extraction

**Created for:**
Radiologists who need consistent, protocol-compliant impressions that follow institutional guidelines and society standards exactly.

---

**Questions? Issues?** Check that:
1. OpenAI API key is valid
2. Documents are .docx format
3. Prompts contain `{document_content}` placeholder
4. Model supports your document size (use gpt-4o for large docs)

# Complete n8n Workflow Guide: Amazon Kindle Book Generator with Google Drive Integration

## 📚 Overview

This guide provides a complete step-by-step tutorial for building an n8n workflow that automatically creates a full, publish-ready Amazon Kindle book and saves it as both DOC and PDF files to Google Drive.

**Perfect for:** Content creators, authors, and publishers who want to automate book creation without coding.

---

## 🎯 What This Workflow Does

1. Takes a book topic (required) and optional outline as input
2. Automatically generates a complete book outline if not provided
3. Creates full book content with proper Kindle-ready formatting
4. Generates front and back cover text suggestions
5. Converts the book into DOC format
6. Converts the book into PDF format
7. Uploads both files to Google Drive automatically
8. Returns confirmation with file links

---

## 📋 Prerequisites

### Required n8n Credentials
- **OpenAI API Key** (or any AI provider like Anthropic, Cohere, etc.)
- **Google Drive OAuth2** credentials

### Google Cloud Setup
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Enable Google Drive API
4. Create OAuth 2.0 credentials
5. Add authorized redirect URI: `https://YOUR_N8N_URL/rest/oauth2-credential/callback`
6. Download credentials (Client ID and Client Secret)

### n8n Setup
1. Install n8n (self-hosted or cloud)
2. Configure AI credentials (OpenAI/Anthropic)
3. Configure Google Drive OAuth2 credentials

---

## 🏗️ Complete Workflow Structure

### Node Flow Diagram

```
[Manual Trigger]
    ↓
[Set Input Parameters]
    ↓
[Check if Outline Exists] (IF Node)
    ↓                    ↓
    YES                  NO
    ↓                    ↓
[Use Provided]    [Generate Outline]
    ↓                    ↓
    └──────┬─────────────┘
           ↓
[Generate Full Book Content]
           ↓
[Generate Front Cover Text]
           ↓
[Generate Back Cover Text]
           ↓
[Merge All Content]
           ↓
    ┌──────┴──────┐
    ↓             ↓
[Create DOC]  [Create PDF]
    ↓             ↓
    └──────┬──────┘
           ↓
[Create GDrive Folder]
           ↓
[Upload DOC to GDrive]
           ↓
[Upload PDF to GDrive]
           ↓
[Return Confirmation]
```

---

## 🔧 Detailed Node Configuration

### Node 1: Manual Trigger
**Purpose:** Start the workflow manually  
**Type:** Manual Trigger node  
**Configuration:**
- No configuration needed
- This allows you to trigger the workflow on demand

**How it works:** Click "Test workflow" or "Execute workflow" to start the process.

---

### Node 2: Set Input Parameters
**Purpose:** Define the book topic and optional outline  
**Type:** Set node  
**Configuration:**

```json
{
  "values": {
    "string": [
      {
        "name": "topic",
        "value": "The Complete Guide to Meditation for Beginners"
      },
      {
        "name": "outline",
        "value": ""
      }
    ]
  }
}
```

**Parameters explained:**
- `topic`: (REQUIRED) The main subject of your book
- `outline`: (OPTIONAL) Leave empty to auto-generate, or provide like:
  ```
  Chapter 1: Introduction to Meditation
  Chapter 2: Breathing Techniques
  Chapter 3: Mindfulness Practices
  Chapter 4: Advanced Meditation
  Chapter 5: Daily Practice Guide
  ```

**How to use:**
1. Add a "Set" node
2. Add a string field named "topic"
3. Add a string field named "outline"
4. Fill in your book topic
5. Optionally fill in the outline

---

### Node 3: Check if Outline Exists
**Purpose:** Determine if we need to generate an outline  
**Type:** IF node  
**Configuration:**

```json
{
  "conditions": {
    "string": [
      {
        "value1": "={{ $json.outline }}",
        "operation": "isNotEmpty"
      }
    ]
  }
}
```

**How it works:**
- Checks if the "outline" field has content
- If YES (outline provided): Goes to TRUE branch → Uses provided outline
- If NO (outline empty): Goes to FALSE branch → Generates outline automatically

**Connection rules:**
- TRUE output → Connect directly to "Generate Full Book Content" node
- FALSE output → Connect to "Generate Outline" node

---

### Node 4: Generate Outline (Only if outline is missing)
**Purpose:** Create a comprehensive book outline automatically  
**Type:** OpenAI/AI node  
**Configuration:**

**Model:** gpt-4 or gpt-3.5-turbo  
**Operation:** Message a model  
**Prompt:**

```
You are a professional book outline creator. Create a detailed outline for a complete Amazon Kindle book on the following topic:

Topic: {{ $json.topic }}

Generate a full book outline with the following structure:
- Book Title (create an engaging title)
- Subtitle (create a descriptive subtitle)
- Introduction
- Table of Contents listing
- Chapter 1 through Chapter 10 (create meaningful chapter titles based on the topic)
- Conclusion
- Author Bio section

Requirements:
- Use plain text only
- No markdown formatting
- No asterisks or special characters
- Each chapter should have a clear, descriptive title
- Output format should be clean and simple
- List each section on a new line

Output the outline with this exact format:
Title: [Book Title]
Subtitle: [Subtitle]
Introduction
Chapter 1: [Chapter Title]
Chapter 2: [Chapter Title]
Chapter 3: [Chapter Title]
Chapter 4: [Chapter Title]
Chapter 5: [Chapter Title]
Chapter 6: [Chapter Title]
Chapter 7: [Chapter Title]
Chapter 8: [Chapter Title]
Chapter 9: [Chapter Title]
Chapter 10: [Chapter Title]
Conclusion
Author Bio
```

**Output:** Assigns generated outline to a variable for next steps  
**Assign to:** `outline`

---

### Node 5: Generate Full Book Content
**Purpose:** Write the complete book content based on the outline  
**Type:** OpenAI/AI node (use GPT-4 for best quality)  
**Configuration:**

**Model:** gpt-4-turbo or gpt-4  
**Operation:** Message a model  
**Max Tokens:** 4000 (or maximum available)  
**Temperature:** 0.7  

**Prompt:**

```
You are a professional book writer creating content for Amazon Kindle Direct Publishing.

Topic: {{ $json.topic }}

Outline to follow:
{{ $node["Check if Outline Exists"].json.outline || $node["Generate Outline"].json.output }}

Write a COMPLETE book with full content for every section listed in the outline above.

CRITICAL REQUIREMENTS:
1. Write FULL detailed content for each chapter (minimum 800 words per chapter)
2. Use plain text ONLY - absolutely no markdown
3. No asterisks, no special characters, no formatting symbols
4. Use clear section headings exactly as shown in the outline
5. Each chapter must have substantive, valuable content
6. Write in a professional, engaging style suitable for Kindle readers
7. Content must be publication-ready

STRUCTURE YOUR OUTPUT EXACTLY LIKE THIS:

[Book Title from outline]

[Subtitle from outline]

Introduction

[Write full introduction content here - minimum 500 words]

Table of Contents

Introduction
Chapter 1: [Title]
Chapter 2: [Title]
[... list all chapters ...]
Conclusion
Author Bio

Chapter 1: [Chapter Title]

[Write complete chapter content here - minimum 800 words]

Chapter 2: [Chapter Title]

[Write complete chapter content here - minimum 800 words]

[Continue for all chapters...]

Conclusion

[Write full conclusion content here - minimum 500 words]

Author Bio

[Write a professional author bio - 200 words]

Remember: Use ONLY plain text. No special formatting. Make it Kindle-ready.
```

**Output parsing:** The full book text  
**Assign to:** `bookContent`

---

### Node 6: Generate Front Cover Text
**Purpose:** Create front cover design suggestions  
**Type:** OpenAI/AI node  
**Configuration:**

**Model:** gpt-3.5-turbo or gpt-4  
**Prompt:**

```
You are a professional book cover designer providing text-only design suggestions.

Book Topic: {{ $json.topic }}

Create front cover text suggestions including:

1. Three alternative book title variations (engaging and marketable)
2. A subtitle suggestion for each title variation
3. Color theme suggestions for each version (describe colors only)
4. Typography style suggestions (describe font style recommendations)

Output ONLY plain text descriptions. No markdown. No special characters.

Format your output like this:

VERSION 1
Title: [Title]
Subtitle: [Subtitle]
Color Theme: [Color description]
Typography Style: [Font style description]

VERSION 2
Title: [Title]
Subtitle: [Subtitle]
Color Theme: [Color description]
Typography Style: [Font style description]

VERSION 3
Title: [Title]
Subtitle: [Subtitle]
Color Theme: [Color description]
Typography Style: [Font style description]
```

**Output:** Assigns suggestions to variable  
**Assign to:** `frontCoverText`

---

### Node 7: Generate Back Cover Text
**Purpose:** Create back cover marketing copy  
**Type:** OpenAI/AI node  
**Configuration:**

**Model:** gpt-3.5-turbo or gpt-4  
**Prompt:**

```
You are a professional book marketing copywriter creating back cover text.

Book Topic: {{ $json.topic }}

Create back cover marketing text including:

1. Three different short persuasive book descriptions (each 100-150 words)
2. One compelling call-to-action line for each description
3. A short generic professional author bio (100 words)

Requirements:
- Use plain text only
- No markdown or special formatting
- No asterisks or symbols
- Make it compelling and persuasive
- Focus on reader benefits
- Suitable for Amazon Kindle

Format your output like this:

DESCRIPTION 1
[Write persuasive description here]

Call to Action: [One compelling line]

DESCRIPTION 2
[Write persuasive description here]

Call to Action: [One compelling line]

DESCRIPTION 3
[Write persuasive description here]

Call to Action: [One compelling line]

AUTHOR BIO
[Write professional generic author bio here]
```

**Output:** Assigns marketing text  
**Assign to:** `backCoverText`

---

### Node 8: Merge All Content
**Purpose:** Combine all generated content into one complete document  
**Type:** Set node or Code node  
**Configuration:**

Using **Set node**:
```json
{
  "values": {
    "string": [
      {
        "name": "completeBook",
        "value": "={{ $node[\"Generate Full Book Content\"].json.bookContent }}\n\n=== FRONT COVER SUGGESTIONS ===\n\n={{ $node[\"Generate Front Cover Text\"].json.frontCoverText }}\n\n=== BACK COVER TEXT ===\n\n={{ $node[\"Generate Back Cover Text\"].json.backCoverText }}"
      },
      {
        "name": "fileName",
        "value": "={{ $json.topic.replace(/[^a-z0-9]/gi, '_').substring(0, 50) }}"
      }
    ]
  }
}
```

Using **Code node** (Alternative - more flexible):
```javascript
// Get content from previous nodes
const bookContent = $node["Generate Full Book Content"].json.bookContent;
const frontCover = $node["Generate Front Cover Text"].json.frontCoverText;
const backCover = $node["Generate Back Cover Text"].json.backCoverText;
const topic = $json.topic;

// Create complete book with all sections
const completeBook = `${bookContent}

=== FRONT COVER SUGGESTIONS ===

${frontCover}

=== BACK COVER TEXT ===

${backCover}`;

// Create safe file name from topic
const fileName = topic
  .replace(/[^a-z0-9]/gi, '_')
  .substring(0, 50);

// Return merged data
return {
  json: {
    completeBook: completeBook,
    fileName: fileName,
    topic: topic
  }
};
```

**How it works:** Concatenates all generated content sections into one variable for file generation.

---

### Node 9: Create DOC File
**Purpose:** Convert text content to Microsoft Word format  
**Type:** Convert to File node (or HTTP Request to document conversion API)  
**Configuration:**

**Option A: Using n8n's built-in conversion (if available):**
- File format: DOCX
- Content: `={{ $json.completeBook }}`
- File name: `={{ $json.fileName }}.docx`

**Option B: Using an API service like CloudConvert:**
1. Add HTTP Request node
2. Method: POST
3. URL: CloudConvert API endpoint
4. Authentication: API Key
5. Body:
   ```json
   {
     "tasks": {
       "import-text": {
         "operation": "import/upload"
       },
       "convert": {
         "input": "import-text",
         "output_format": "docx"
       },
       "export": {
         "input": "convert"
       }
     }
   }
   ```

**Option C: Using Google Docs API (Recommended for Google integration):**
1. Use HTTP Request node
2. Method: POST
3. URL: `https://docs.googleapis.com/v1/documents`
4. Authentication: OAuth2 (Google)
5. Body: Create document with title and content

**Simplified approach using File node:**
```json
{
  "operation": "toBinaryData",
  "options": {
    "mimeType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
    "fileName": "={{ $json.fileName }}.docx",
    "dataPropertyName": "data"
  }
}
```

---

### Node 10: Create PDF File
**Purpose:** Convert text content to PDF format  
**Type:** Convert to File node or HTTP Request to conversion API  
**Configuration:**

**Using an online conversion service (recommended):**

1. Add HTTP Request node
2. Method: POST
3. Service: PDFShift, CloudConvert, or similar
4. URL: Service API endpoint
5. Headers:
   ```json
   {
     "Content-Type": "application/json",
     "Authorization": "Basic YOUR_API_KEY"
   }
   ```
6. Body:
   ```json
   {
     "source": "={{ $json.completeBook }}",
     "filename": "={{ $json.fileName }}.pdf",
     "format": "A4",
     "margin": {
       "top": "1in",
       "bottom": "1in",
       "left": "1in",
       "right": "1in"
     }
   }
   ```

**Alternative using wkhtmltopdf (self-hosted):**
```javascript
// Code node to prepare HTML
const content = $json.completeBook;
const html = `
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; margin: 40px; }
        h1 { page-break-before: always; }
    </style>
</head>
<body>
    <pre style="white-space: pre-wrap; font-family: Arial;">${content}</pre>
</body>
</html>
`;

return { json: { html: html, fileName: $json.fileName } };
```

Then use Execute Command node:
```bash
echo '{{ $json.html }}' | wkhtmltopdf - /tmp/{{ $json.fileName }}.pdf
```

---

### Node 11: Create Google Drive Folder
**Purpose:** Create a dedicated folder for the book files  
**Type:** Google Drive node  
**Configuration:**

**Operation:** Create a folder  
**Folder Name:** `{{ $json.topic }} - Kindle Book`  
**Parent Folder:** Root (or specify a folder ID)  
**Options:**
- If folder exists: Use existing folder

**Settings:**
```json
{
  "operation": "create",
  "resource": "folder",
  "name": "={{ $json.topic }} - Kindle Book",
  "options": {
    "parents": []
  }
}
```

**Output:** This provides the folder ID for uploading files

---

### Node 12: Upload DOC to Google Drive
**Purpose:** Upload the Word document to Google Drive  
**Type:** Google Drive node  
**Configuration:**

**Operation:** Upload  
**File Name:** `={{ $json.fileName }}.docx`  
**Binary Data:** Yes (from Node 9 output)  
**Parent Folder ID:** `={{ $node["Create Google Drive Folder"].json.id }}`  
**Convert to Google Docs:** Optional (recommended for easier editing)

**Settings:**
```json
{
  "operation": "upload",
  "resource": "file",
  "name": "={{ $json.fileName }}.docx",
  "binaryData": true,
  "parents": {
    "parent": [
      {
        "id": "={{ $node[\"Create Google Drive Folder\"].json.id }}"
      }
    ]
  },
  "options": {
    "convert": false
  }
}
```

**Output:** Returns file ID and web link

---

### Node 13: Upload PDF to Google Drive
**Purpose:** Upload the PDF document to Google Drive  
**Type:** Google Drive node  
**Configuration:**

**Operation:** Upload  
**File Name:** `={{ $json.fileName }}.pdf`  
**Binary Data:** Yes (from Node 10 output)  
**Parent Folder ID:** `={{ $node["Create Google Drive Folder"].json.id }}`

**Settings:**
```json
{
  "operation": "upload",
  "resource": "file",
  "name": "={{ $json.fileName }}.pdf",
  "binaryData": true,
  "parents": {
    "parent": [
      {
        "id": "={{ $node[\"Create Google Drive Folder\"].json.id }}"
      }
    ]
  }
}
```

**Output:** Returns file ID and web link

---

### Node 14: Return Confirmation
**Purpose:** Provide final output with links to files  
**Type:** Set node  
**Configuration:**

```json
{
  "values": {
    "string": [
      {
        "name": "status",
        "value": "SUCCESS"
      },
      {
        "name": "message",
        "value": "Your Kindle book has been created successfully!"
      },
      {
        "name": "topic",
        "value": "={{ $json.topic }}"
      },
      {
        "name": "docLink",
        "value": "={{ $node[\"Upload DOC to Google Drive\"].json.webViewLink }}"
      },
      {
        "name": "pdfLink",
        "value": "={{ $node[\"Upload PDF to Google Drive\"].json.webViewLink }}"
      },
      {
        "name": "folderLink",
        "value": "={{ $node[\"Create Google Drive Folder\"].json.webViewLink }}"
      }
    ]
  }
}
```

**Output example:**
```json
{
  "status": "SUCCESS",
  "message": "Your Kindle book has been created successfully!",
  "topic": "The Complete Guide to Meditation for Beginners",
  "docLink": "https://docs.google.com/document/d/...",
  "pdfLink": "https://drive.google.com/file/d/...",
  "folderLink": "https://drive.google.com/drive/folders/..."
}
```

---

## 🔗 Node Connections Summary

Here's how all nodes connect to each other:

```
1. Manual Trigger → Set Input Parameters
2. Set Input Parameters → Check if Outline Exists
3. Check if Outline Exists (FALSE) → Generate Outline
4. Check if Outline Exists (TRUE) → Generate Full Book Content
5. Generate Outline → Generate Full Book Content
6. Generate Full Book Content → Generate Front Cover Text
7. Generate Front Cover Text → Generate Back Cover Text
8. Generate Back Cover Text → Merge All Content
9. Merge All Content → Create DOC File
10. Merge All Content → Create PDF File
11. Create DOC File → Create Google Drive Folder
12. Create Google Drive Folder → Upload DOC to Google Drive
13. Upload DOC to Google Drive → Upload PDF to Google Drive
14. Upload PDF to Google Drive → Return Confirmation
```

---

## 📝 How to Use This Workflow

### For Non-Technical Users:

1. **Open your n8n instance** (web browser)

2. **Create a new workflow** (click "+ New Workflow")

3. **Add nodes** by clicking the "+" button and following the structure above

4. **Configure credentials:**
   - Add OpenAI credentials (Settings → Credentials → New → OpenAI)
   - Add Google Drive credentials (Settings → Credentials → New → Google Drive OAuth2)

5. **Set up each node** using the configurations provided above

6. **Save your workflow**

7. **Test it:**
   - Click "Execute Workflow" button
   - The workflow will prompt for topic input (or use the Set node values)
   - Wait for execution to complete
   - Check the final node for Google Drive links

8. **Access your book:**
   - Open the folder link from the output
   - Download DOC and PDF files
   - Upload to Amazon Kindle Direct Publishing

---

## 🎨 Customization Options

### Adjust Book Length
In Node 5 (Generate Full Book Content), modify the prompt:
- Change "minimum 800 words per chapter" to your preferred length
- Adjust number of chapters in the outline

### Change AI Provider
Switch from OpenAI to:
- **Anthropic Claude:** Better for longer content
- **Google PaLM:** Good for specific topics
- **Cohere:** Cost-effective option

### Modify Book Structure
Edit the outline generation prompt to include:
- More or fewer chapters
- Additional sections (foreword, appendix, glossary)
- Different formatting requirements

### Add Quality Checks
Insert additional nodes:
- **Grammar check** (using LanguageTool API)
- **Plagiarism check** (using Copyscape API)
- **Readability score** (using Hemingway API)

---

## 🐛 Troubleshooting

### Issue: "Outline not generating"
**Solution:**
- Check OpenAI API key is valid
- Ensure you have sufficient API credits
- Verify the prompt is correctly formatted
- Try reducing prompt complexity

### Issue: "Files not uploading to Google Drive"
**Solution:**
- Re-authenticate Google Drive credentials
- Check folder permissions
- Verify binary data is correctly passed from conversion nodes
- Test Google Drive connection with a simple upload

### Issue: "PDF/DOC conversion fails"
**Solution:**
- Ensure conversion service API key is valid
- Check content doesn't exceed size limits
- Verify MIME types are correctly set
- Test with shorter content first

### Issue: "Workflow times out"
**Solution:**
- Increase timeout settings in n8n
- Split book generation into smaller chunks
- Use async execution for long-running AI tasks
- Reduce max tokens per AI request

### Issue: "Content has formatting issues"
**Solution:**
- Review AI prompts to emphasize plain text
- Add post-processing node to strip unwanted characters
- Use regex to clean content before file conversion

---

## 📊 Expected Execution Time

- **Outline generation:** 10-30 seconds
- **Full book content:** 2-5 minutes (depending on length and AI model)
- **Front/back cover text:** 20-40 seconds each
- **File conversion:** 10-20 seconds each
- **Google Drive upload:** 5-10 seconds each

**Total time:** Approximately 4-7 minutes for complete workflow

---

## 💰 Estimated Costs

### AI API Costs (OpenAI GPT-4):
- Outline generation: ~$0.05-$0.10
- Full book content: ~$0.50-$2.00 (depending on length)
- Cover text generation: ~$0.05-$0.10 each

**Total per book:** $0.65-$2.30

### Using GPT-3.5-Turbo (cheaper):
- Total per book: ~$0.10-$0.30

### Document Conversion (if using paid service):
- CloudConvert: ~$0.01-$0.05 per conversion

### Google Drive:
- Free (15GB storage limit on free tier)

---

## ✅ Testing Checklist

Before using in production, test:

- [ ] Manual trigger works
- [ ] Input parameters are correctly set
- [ ] Outline condition properly checks for content
- [ ] Auto-generated outline is comprehensive
- [ ] Book content is complete and well-formatted
- [ ] Front cover suggestions are creative
- [ ] Back cover text is persuasive
- [ ] Content merging includes all sections
- [ ] DOC file is properly formatted
- [ ] PDF file is readable
- [ ] Google Drive folder is created
- [ ] DOC file uploads successfully
- [ ] PDF file uploads successfully
- [ ] Final confirmation includes all links
- [ ] Links are accessible and files downloadable

---

## 🚀 Advanced Enhancements

### Add Email Notification
After Node 14, add:
- **Gmail/Send Email node**
- Send confirmation email with download links
- Include book preview in email body

### Create Multiple Book Formats
Add parallel branches after content generation:
- EPUB format (for other platforms)
- MOBI format (legacy Kindle)
- Plain text file

### Implement Version Control
- Add timestamp to file names
- Keep previous versions in separate folder
- Create changelog document

### Add Human Review Step
- Insert "Wait" node after content generation
- Send for manual review via Slack/Teams
- Continue only after approval

### Batch Processing
- Replace Manual Trigger with Webhook
- Accept multiple topics via CSV
- Process books in queue
- Generate multiple books automatically

---

## 📖 Example Usage Scenario

**Input:**
```
Topic: "Mindful Eating: A 30-Day Guide to Better Health"
Outline: (left empty)
```

**Workflow execution:**
1. Detects no outline provided
2. Generates 10-chapter outline about mindful eating
3. Writes complete book content (~15,000 words)
4. Creates 3 title variations with cover design suggestions
5. Generates marketing copy for back cover
6. Merges all content
7. Converts to DOC format
8. Converts to PDF format
9. Creates folder "Mindful Eating - Kindle Book" in Google Drive
10. Uploads both files
11. Returns success message with links

**Output:**
```json
{
  "status": "SUCCESS",
  "message": "Your Kindle book has been created successfully!",
  "topic": "Mindful Eating: A 30-Day Guide to Better Health",
  "docLink": "https://docs.google.com/document/d/abc123...",
  "pdfLink": "https://drive.google.com/file/d/xyz789...",
  "folderLink": "https://drive.google.com/drive/folders/folder123..."
}
```

**Time:** ~5 minutes  
**Cost:** ~$1.50 (using GPT-4)

---

## 🔐 Security Best Practices

1. **Protect API Keys:**
   - Never hardcode API keys
   - Use n8n credential system
   - Rotate keys regularly

2. **Google Drive Permissions:**
   - Use least-privilege access
   - Create dedicated folder for book files
   - Review shared access regularly

3. **Content Privacy:**
   - Don't send sensitive topics to AI
   - Review AI provider data retention policies
   - Consider self-hosted AI models for private content

4. **Workflow Access:**
   - Restrict workflow execution permissions
   - Use webhook authentication if exposing publicly
   - Monitor execution logs

---

## 📚 Additional Resources

### n8n Documentation
- [n8n Official Docs](https://docs.n8n.io/)
- [OpenAI Node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.openai/)
- [Google Drive Node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googledrive/)

### Amazon Kindle Publishing
- [Kindle Direct Publishing](https://kdp.amazon.com/)
- [KDP Formatting Guidelines](https://kdp.amazon.com/en_US/help/topic/G200645680)
- [Kindle Create Tool](https://www.amazon.com/Kindle-Create/b?ie=UTF8&node=18292298011)

### Document Conversion APIs
- [CloudConvert](https://cloudconvert.com/)
- [PDFShift](https://pdfshift.io/)
- [ConvertAPI](https://www.convertapi.com/)

---

## 🎓 Learning Path for n8n Beginners

1. **Week 1: Basics**
   - Install n8n
   - Create simple workflows
   - Understand nodes and connections

2. **Week 2: Data Handling**
   - Learn expressions `={{ }}`
   - Work with JSON data
   - Use Set and Code nodes

3. **Week 3: APIs and Integrations**
   - Configure credentials
   - Connect to external services
   - Handle API responses

4. **Week 4: Advanced Workflows**
   - Implement conditional logic
   - Use loops and iterations
   - Build this Kindle book workflow!

---

## 🤝 Support and Community

### Get Help
- [n8n Community Forum](https://community.n8n.io/)
- [n8n Discord](https://discord.gg/n8n)
- [Stack Overflow - n8n tag](https://stackoverflow.com/questions/tagged/n8n)

### Share Your Books
Once you've created books with this workflow:
- Share your success stories
- Provide feedback on the workflow
- Contribute improvements

---

## 📄 License and Usage

This workflow guide is provided as-is for educational and commercial use. Feel free to:
- Modify for your needs
- Share with others
- Use in production
- Sell books created with it

**Attribution appreciated but not required.**

---

## 🎯 Next Steps

1. **Set up n8n** (if not already done)
2. **Configure credentials** (OpenAI, Google Drive)
3. **Build the workflow** following this guide
4. **Test with a simple topic** first
5. **Refine prompts** based on your needs
6. **Create your first book!**
7. **Upload to Amazon KDP**
8. **Start publishing!**

---

**Happy Book Creating! 📚✨**

For questions or improvements to this guide, please open an issue or pull request.

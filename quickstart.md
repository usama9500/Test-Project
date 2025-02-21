# **Quick Start Guide: APIMatic Docs as Code**  

This guide will help you quickly set up and generate an API portal using APIMatic’s **Docs as Code** approach. Whether you are new to Docs as Code or looking for a streamlined setup, follow these steps to get started.

---

## **Step 1: Prepare the Build Input**  
Before generating an API portal, you need to set up a **Docs as Code repository** that includes your API specification, documentation, and configuration.

### **1. Create a Repository**  
You can fork a sample repository or create a new one. If you are setting up a new repository, structure it as follows:

```
docs-as-code-repo/
├── spec/           # API specification files (OpenAPI, RAML, Postman, etc.)
├── content/        # Markdown documentation (guides, changelogs, etc.)
├── static/         # Static assets (images, logos, PDFs, etc.)
├── APIMATIC-BUILD.json  # Configuration file for API portal generation
```

### **2. Add Your API Specification**  
Place your **OpenAPI, RAML, API Blueprint, or Postman** collection in the `spec/` directory. APIMatic will **automatically detect the format** and generate documentation accordingly.

### **3. Add Custom Documentation (Optional)**  
You can include additional guides by adding Markdown files to the `content/` directory. Define a **table of contents** using `toc.yml`:

```yaml
toc:
  - group: Getting Started
    items:
      - generate: Introduction
        from: intro
  - group: API Reference
    items:
      - generate: Endpoints
        from: endpoints
```

### **4. Configure the Build File**  
The **APIMATIC-BUILD.json** file specifies how the API portal is generated. A minimal example:

```json
{
  "$schema": "https://titan.apimatic.io/api/build/schema",
  "buildFileVersion": "1",
  "generatePortal": {
     "apiSpecs": ["spec/openapi.json"],
     "languageConfig": {
       "http": {}
     }
  }
}
```

---

## **Step 2: Generate an API Portal Using Docs as Code Workflow**  
Once your repository is set up, generate an **API Portal** using **APIMatic’s API**.

### **Option 1: Generate Using the Sync API**  
For small API specs, use the **Sync API**:

```bash
curl -X POST "https://api.apimatic.io/v1/generate-portal" \
-H "Authorization: Bearer YOUR_API_KEY" \
-H "Content-Type: application/json" \
-d '{
    "buildInput": {
        "apiSpecs": ["spec/openapi.json"],
        "languageConfig": {
            "http": {}
        }
    }
}' --output portal.zip
```

### **Option 2: Generate Using the Async API**  
For large API specifications, use the **Async API** to avoid timeouts:

```bash
curl -X POST "https://api.apimatic.io/v1/generate-portal-async" \
-H "Authorization: Bearer YOUR_API_KEY" \
-H "Content-Type: application/json" \
-d '{
    "buildInput": {
        "apiSpecs": ["spec/openapi.json"],
        "languageConfig": {
            "http": {}
        }
    }
}'
```

This returns a **status URL**. Poll the endpoint until the portal is ready, then download it.

---

## **Step 3: Automate API Portal Generation with GitHub Actions**  
To keep your API documentation up to date, automate the process with **GitHub Actions**.

### **1. Set Up Repository Secrets**  
- `API_KEY` – Your APIMatic API key  
- `NETLIFY_AUTH_TOKEN` – Netlify authentication token  
- `NETLIFY_SITE_ID` – Netlify site ID  

### **2. Add a GitHub Actions Workflow (`.github/workflows/deploy.yml`)**  

```yaml
name: Generate and Deploy API Portal

on:
  push:
    branches:
      - main

jobs:
  generate-portal:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Call APIMatic API
        run: curl -X POST "https://api.apimatic.io/v1/generate-portal" \
               -H "Authorization: Bearer ${{ secrets.API_KEY }}" \
               -H "Content-Type: application/json" \
               -o portal.zip
      - name: Extract and Deploy
        run: unzip -qq portal.zip -d static-portal
      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v2.0
        with:
          publish-dir: './static-portal'
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

Now, every time you push changes to your repository, your API portal will **automatically regenerate and deploy**.

---

## **Step 4: Migrating from an Existing Portal to the Docs as Code Workflow**  
If you are currently using APIMatic’s **UI-based workflow**, you can migrate to Docs as Code by **exporting a Build Input file**.

### **1. Retrieve API Integration Keys**  
1. Go to **APIMatic Dashboard**  
2. Select the API you want to migrate  
3. Click on the **three-dot menu** → **API Integration Keys**  
4. Copy the **API Group ID**  

### **2. Generate Build Input from an Existing Portal**  
Use the following API request to generate a **Build Input** file:  

```bash
curl -X GET "https://api.apimatic.io/v1/portal/build-input?groupId=YOUR_API_GROUP_ID" \
-H "Authorization: Bearer YOUR_API_KEY" \
-H "Accept: application/json" -o build-input.zip
```

### **3. Use Build Input in Your Docs as Code Repository**  
Extract the `build-input.zip` file and place its contents into your **Docs as Code** repository, ensuring everything is structured correctly.

### **4. Regenerate the API Portal Using the Docs as Code Workflow**  
Once you have the **Build Input**, follow the earlier steps to generate the API Portal using either the **Sync or Async API**.

---

## **Step 5: Hosting Your API Portal**  
Once the API portal is generated, you need to host it.

### **1. Preview Locally**  
Extract the ZIP file and serve it locally:

```bash
unzip portal.zip -d portal
npx http-server ./portal
```

Then open `http://localhost:8080` to preview.

### **2. Deploy to GitHub Pages**  
For GitHub Pages, use this GitHub Action:

```yaml
- name: Deploy to GitHub Pages
  uses: peaceiris/actions-gh-pages@v3
  with:
    publish_dir: ./portal
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

### **3. Deploy to Netlify (GitHub Actions)**  
If you are using GitHub Actions, deploy the portal to Netlify with the following workflow:

```yaml
- name: Deploy to Netlify
  uses: nwtgck/actions-netlify@v2.0
  with:
    publish-dir: './portal'
    github-token: ${{ secrets.GITHUB_TOKEN }}
  env:
    NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
    NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

---

## **Next Steps**  
- Start using Docs as Code by setting up your repository.  
- Read the full **APIMatic API Documentation** for more details.  
- Automate portal generation in your CI/CD pipelines for seamless updates.  

This **quick start guide** provides the essential steps to help you set up Docs as Code efficiently and get your API portal running.

# Lecture 7 – Documentation with Pandoc (PDF)

### Goals
- Generate a **PDF version** of the existing MkDocs documentation
- Integrate PDF generation into the CI/CD pipeline **before** building the documentation image
- Serve the PDF file within the documentation container

---

### Setup
- MkDocs builds HTML site in `docs/build/site`
- Pandoc converts the documentation to PDF and saves it in the same folder:
  ```
  docs/build/site/DevOps_Lecture7_Documentation.pdf
  ```

---

### Pipeline Integration
1. **Generate PDF** using Pandoc:
   ```yaml
   - name: Generate PDF with Pandoc
     run: |
       docker run --rm -v "$PWD":/data pandoc/latex:latest          -s docs/content/index.md -o docs/build/site/documentation.pdf
   ```

2. **Build Nginx image** including the PDF:
   ```dockerfile
   FROM nginx
   COPY docs/build/site /usr/share/nginx/html
   ```

---

### Result
**Lecture 7:** The documentation site now includes a downloadable PDF version (`DevOps_Lecture7_Documentation.pdf`), built automatically in the pipeline and served by the Nginx container.

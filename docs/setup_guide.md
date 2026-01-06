# Setup Guide - Ship Order Knowledge Graph
## Complete Step-by-Step Instructions for Beginners

This guide will walk you through setting up and running the Ship Order Knowledge Graph from scratch. Every step is explained in detail, so you don't need any prior experience.

---

## Table of Contents
1. [What You Need](#what-you-need)
2. [Step 1: Install Java](#step-1-install-java)
3. [Step 2: Download Apache Jena Fuseki](#step-2-download-apache-jena-fuseki)
4. [Step 3: Set Up Fuseki Server](#step-3-set-up-fuseki-server)
5. [Step 4: Load the Data](#step-4-load-the-data)
6. [Step 5: Run Your First Query](#step-5-run-your-first-query)
7. [Troubleshooting](#troubleshooting)
8. [Next Steps](#next-steps)

---

## What You Need

Before we start, here's what you'll need:

1. **A computer** (Windows, Mac, or Linux)
2. **Internet connection** (to download software)
3. **About 30 minutes** of your time
4. **Basic computer skills** (clicking, typing, opening files)

**What you DON'T need:**
- Programming experience
- Special software already installed
- A powerful computer (any modern computer will work)

---

## Step 1: Install Java

Java is a programming language that our SPARQL server needs to run.

### For Windows:

1. **Open your web browser** (Chrome, Firefox, Edge, etc.)

2. **Go to this website**: https://www.oracle.com/java/technologies/downloads/

3. **Find the section that says "Windows"** and click on the latest version (it will say something like "Java 17" or "Java 21")

4. **Click the download button** for "Windows x64 Installer" (if you have a 64-bit computer, which most modern computers are)

5. **Wait for the file to download** (it will be in your Downloads folder)

6. **Double-click the downloaded file** (it will be named something like `jdk-21_windows-x64_bin.exe`)

7. **Follow the installation wizard**:
   - Click "Next" on the first screen
   - Click "Next" on the second screen (it will show where Java will be installed)
   - Wait for the installation to complete
   - Click "Close" when it's done

8. **Verify Java is installed**:
   - Press the Windows key + R
   - Type `cmd` and press Enter
   - A black window will appear
   - Type: `java -version`
   - Press Enter
   - You should see something like "java version 21.0.1" or similar
   - If you see this, Java is installed correctly! ✅
   - Type `exit` and press Enter to close the window

### For Mac:

1. **Open your web browser**

2. **Go to this website**: https://www.oracle.com/java/technologies/downloads/

3. **Find the section that says "macOS"** and click on the latest version

4. **Click the download button** for "macOS Installer" (choose the .dmg file)

5. **Wait for the file to download**

6. **Double-click the downloaded file** (it will be in your Downloads folder)

7. **Double-click the installer package** inside the window that opens

8. **Follow the installation wizard**:
   - Click "Continue"
   - Click "Install"
   - Enter your password when asked
   - Wait for installation to complete
   - Click "Close"

9. **Verify Java is installed**:
   - Open "Terminal" (search for it in Spotlight or find it in Applications > Utilities)
   - Type: `java -version`
   - Press Enter
   - You should see version information
   - If you see this, Java is installed correctly! ✅

### For Linux:

1. **Open Terminal** (usually found in Applications or press Ctrl+Alt+T)

2. **Type this command** and press Enter:
   ```bash
   sudo apt update
   ```

3. **Type this command** and press Enter:
   ```bash
   sudo apt install default-jdk -y
   ```

4. **Enter your password** when asked and wait for installation

5. **Verify Java is installed**:
   - Type: `java -version`
   - Press Enter
   - You should see version information
   - If you see this, Java is installed correctly! ✅

---

## Step 2: Download Apache Jena Fuseki

Apache Jena Fuseki is the SPARQL server we'll use to run our queries.

### For All Operating Systems:

1. **Open your web browser**

2. **Go to this website**: https://jena.apache.org/download/

3. **Scroll down** until you see "Apache Jena Fuseki"

4. **Click on the latest version** (it will say something like "4.9.0" or similar)

5. **On the download page, find the section that says "Binary Distribution"**

6. **Click the download link** for the ZIP file:
   - For Windows: `apache-jena-fuseki-4.9.0.zip`
   - For Mac/Linux: `apache-jena-fuseki-4.9.0.zip` (same file works for both)

7. **Wait for the file to download** (it will be in your Downloads folder)

8. **Extract the ZIP file**:
   - **Windows**: Right-click the ZIP file → "Extract All" → Choose a location (like `C:\` or your Documents folder) → Click "Extract"
   - **Mac**: Double-click the ZIP file (it will automatically extract)
   - **Linux**: Right-click → "Extract Here" or use: `unzip apache-jena-fuseki-4.9.0.zip`

9. **Remember where you extracted it** - you'll need this location in the next step!

   **Good locations to extract:**
   - Windows: `C:\apache-jena-fuseki-4.9.0\`
   - Mac: `~/apache-jena-fuseki-4.9.0/` (in your home folder)
   - Linux: `~/apache-jena-fuseki-4.9.0/` (in your home folder)

---

## Step 3: Set Up Fuseki Server

Now we'll start the Fuseki server so we can load our data and run queries.

### For Windows:

1. **Open File Explorer** (the folder icon on your taskbar)

2. **Navigate to where you extracted Fuseki** (for example, `C:\apache-jena-fuseki-4.9.0\`)

3. **Open the folder** - you should see files like:
   - `fuseki-server.bat` (this is what we need!)
   - `fuseki-server.jar`
   - `fuseki` folder
   - Other files

4. **Double-click on `fuseki-server.bat`**

5. **A black window will appear** - this is the server starting up!

6. **Wait until you see a message** that says something like:
   ```
   [2024-01-15 10:30:45] Server     INFO  Started 2024/01/15 10:30:45 on port 3030
   ```

7. **Keep this window open!** This is your server running. Don't close it.

8. **Open your web browser**

9. **Go to this address**: http://localhost:3030

10. **You should see the Fuseki web interface!** 🎉
    - If you see a page with "Fuseki Server" at the top, you're all set!
    - If you see an error, check the Troubleshooting section below

### For Mac:

1. **Open Finder**

2. **Navigate to where you extracted Fuseki** (for example, `~/apache-jena-fuseki-4.9.0/`)

3. **Open the folder** - you should see files like:
   - `fuseki-server` (this is what we need!)
   - `fuseki-server.jar`
   - `fuseki` folder
   - Other files

4. **Open Terminal** (Applications > Utilities > Terminal)

5. **Navigate to the Fuseki folder**:
   ```bash
   cd ~/apache-jena-fuseki-4.9.0
   ```
   (Replace `~/apache-jena-fuseki-4.9.0` with your actual path if different)

6. **Make the script executable**:
   ```bash
   chmod +x fuseki-server
   ```

7. **Start the server**:
   ```bash
   ./fuseki-server
   ```

8. **Wait until you see a message** that says something like:
   ```
   [2024-01-15 10:30:45] Server     INFO  Started 2024/01/15 10:30:45 on port 3030
   ```

9. **Keep this Terminal window open!** This is your server running. Don't close it.

10. **Open your web browser**

11. **Go to this address**: http://localhost:3030

12. **You should see the Fuseki web interface!** 🎉
    - If you see a page with "Fuseki Server" at the top, you're all set!
    - If you see an error, check the Troubleshooting section below

### For Linux:

1. **Open Terminal**

2. **Navigate to where you extracted Fuseki**:
   ```bash
   cd ~/apache-jena-fuseki-4.9.0
   ```
   (Replace `~/apache-jena-fuseki-4.9.0` with your actual path if different)

3. **Make the script executable**:
   ```bash
   chmod +x fuseki-server
   ```

4. **Start the server**:
   ```bash
   ./fuseki-server
   ```

5. **Wait until you see a message** that says something like:
   ```
   [2024-01-15 10:30:45] Server     INFO  Started 2024/01/15 10:30:45 on port 3030
   ```

6. **Keep this Terminal window open!** This is your server running. Don't close it.

7. **Open your web browser**

8. **Go to this address**: http://localhost:3030

9. **You should see the Fuseki web interface!** 🎉
    - If you see a page with "Fuseki Server" at the top, you're all set!
    - If you see an error, check the Troubleshooting section below

---

## Step 4: Load the Data

Now we need to create a dataset and load our knowledge graph data into Fuseki.

### 4.1: Create a New Dataset

1. **In the Fuseki web interface** (http://localhost:3030), you should see a section that says "Add new dataset" or "Manage datasets"

2. **Find the text box** that says "Dataset name" or similar

3. **Type a name** for your dataset: `ships`

4. **Click the button** that says "Create dataset" or "Add dataset"

5. **You should see your new dataset appear!** It might say "ships" in a list

### 4.2: Load the Ontology Files

We need to load three ontology files first. These define the structure of our data.

1. **Click on your dataset** (it should say "ships" or have a link to it)

2. **You should see a page** with tabs like "Query", "Upload", "Info", etc.

3. **Click on the "Upload" tab**

4. **Click "Choose File" or "Browse"**

5. **Navigate to your project folder** (where you downloaded this knowledge graph project)

6. **Go to the `ontology` folder**

7. **Select the file**: `naval_core.owl.ttl`

8. **Click "Upload" or "Send"**

9. **Wait for it to finish** - you should see a success message

10. **Repeat steps 4-9 for these files**:
    - `naval_events.owl.ttl` (in the `ontology` folder)
    - `naval_prov.owl.ttl` (in the `ontology` folder, if it exists)

### 4.3: Load the Taxonomy Files

1. **Still in the "Upload" tab**

2. **Click "Choose File" again**

3. **Navigate to the `taxonomy` folder**

4. **Select and upload these files one by one**:
    - `ship_type.skos.ttl`
    - `mission.skos.ttl`
    - `status.skos.ttl`
    - `navy_branch.skos.ttl` (if it exists)
    - `theater_fleet.skos.ttl` (if it exists)

5. **Wait for each upload to complete**

### 4.4: Load the Ship Data

Now we need to load the actual ship data. This might require using RML mappings (see below) or loading pre-generated RDF files.

**Option A: If you have RDF files ready:**

1. **In the "Upload" tab**

2. **Click "Choose File"**

3. **Navigate to where your RDF data files are** (might be in `data/rdf/` or similar)

4. **Select and upload all RDF files** (they might be named like `nvr_data.ttl`, `wikidata_data.ttl`, etc.)

5. **Wait for uploads to complete**

**Option B: If you need to generate RDF from source data (using RML):**

This is more advanced. You'll need to:

1. **Install RMLMapper**:
   - Download from: https://github.com/RMLio/rmlmapper-java/releases
   - Extract the ZIP file
   - Remember where you put it

2. **Open a new Terminal/Command Prompt** (keep Fuseki running in the other window!)

3. **Navigate to your project folder**

4. **Run the RML mapping**:
   ```bash
   java -jar path/to/rmlmapper.jar -m mappings/nvr.rml.ttl -o data/rdf/nvr_data.ttl
   ```
   (Replace `path/to/rmlmapper.jar` with your actual path)

5. **Repeat for other mappings**:
   ```bash
   java -jar path/to/rmlmapper.jar -m mappings/wikidata_extract.rml.ttl -o data/rdf/wikidata_data.ttl
   ```

6. **Then upload the generated RDF files** using Option A above

### 4.5: Verify Data is Loaded

1. **Click on the "Query" tab** in Fuseki

2. **You should see a text area** where you can type SPARQL queries

3. **Type this simple query** to count how many ships you have:
   ```sparql
   PREFIX ship: <https://example.org/ship-ontology/>
   
   SELECT (COUNT(?s) AS ?count) WHERE {
       ?s a ship:Ship .
   }
   ```

4. **Click "Run Query" or press Ctrl+Enter**

5. **You should see a result** showing a number (like "10" or "50" depending on your data)

6. **If you see a number, your data is loaded!** ✅
   - If you see "0", the data might not be loaded correctly
   - If you see an error, check the Troubleshooting section

---

## Step 5: Run Your First Query

Now let's run one of the demo queries to see the knowledge graph in action!

1. **Make sure you're on the "Query" tab** in Fuseki

2. **Open the file** `queries/demo_queries.sparql` in a text editor (Notepad, TextEdit, VS Code, etc.)

3. **Find Query 1** - it should start with:
   ```sparql
   # QUERY 1: Active Ships by Navy
   ```

4. **Copy the entire query** (from `SELECT` to the end, including all the `WHERE` clause)

5. **Paste it into the query text area** in Fuseki

6. **Click "Run Query" or press Ctrl+Enter**

7. **You should see a table** with columns like:
   - `operatorName`
   - `shipName`
   - `hullSymbol`
   - `class`
   - `homeportName`

8. **Each row represents one active ship!** 🎉

9. **Try other queries**:
   - Copy Query 2, 3, 4, etc. from the file
   - Paste and run them
   - See what results you get!

---

## Troubleshooting

### Problem: "Java is not recognized"

**Solution:**
- Make sure Java is installed (go back to Step 1)
- On Windows, you might need to restart your computer after installing Java
- On Mac/Linux, make sure Java is in your PATH

### Problem: "Port 3030 is already in use"

**Solution:**
- Another program is using port 3030
- Close any other Fuseki servers that might be running
- Or change the port: add `--port=3031` when starting Fuseki

### Problem: "Cannot connect to localhost:3030"

**Solution:**
- Make sure Fuseki is still running (check the Terminal/Command Prompt window)
- Make sure you typed `http://localhost:3030` (not `https://`)
- Try closing and restarting Fuseki

### Problem: "Query returns 0 results"

**Solution:**
- Make sure you uploaded all the data files (go back to Step 4)
- Check that the ontology files were uploaded first
- Verify the data files are in the correct format (Turtle .ttl files)
- Try a simpler query first to test:
  ```sparql
  SELECT * WHERE { ?s ?p ?o } LIMIT 10
  ```

### Problem: "Error loading file"

**Solution:**
- Make sure the file path is correct
- Make sure the file is a valid Turtle (.ttl) or RDF file
- Check that the file isn't corrupted
- Try uploading a smaller file first to test

### Problem: "Out of memory" error

**Solution:**
- Fuseki might need more memory
- When starting Fuseki, add: `-Xmx2G` (for 2GB of memory)
- Example: `java -Xmx2G -jar fuseki-server.jar`

---

## Next Steps

Congratulations! You've successfully set up the Ship Order Knowledge Graph! 🎉

### What to do next:

1. **Explore the demo queries**:
   - Open `queries/demo_queries.sparql`
   - Try running different queries
   - See what insights you can discover!

2. **Read the documentation**:
   - `docs/demo_script.md` - Learn about the queries
   - `docs/scope.md` - Understand the project scope
   - `docs/competency_questions.md` - See what questions the system can answer

3. **Experiment**:
   - Write your own SPARQL queries
   - Try different combinations
   - See what questions you can answer!

4. **Learn more**:
   - SPARQL Tutorial: https://www.w3.org/TR/sparql11-query/
   - Fuseki Documentation: https://jena.apache.org/documentation/fuseki2/
   - RDF Basics: https://www.w3.org/RDF/

---

## Quick Reference

### Starting Fuseki:

**Windows:**
```
cd C:\apache-jena-fuseki-4.9.0
fuseki-server.bat
```

**Mac/Linux:**
```bash
cd ~/apache-jena-fuseki-4.9.0
./fuseki-server
```

### Accessing Fuseki:
- Web Interface: http://localhost:3030
- SPARQL Endpoint: http://localhost:3030/ships/sparql

### Important Files:
- Ontologies: `ontology/*.owl.ttl`
- Taxonomies: `taxonomy/*.skos.ttl`
- Queries: `queries/demo_queries.sparql`
- Mappings: `mappings/*.rml.ttl`

---

## Need Help?

If you're stuck:
1. Check the Troubleshooting section above
2. Read the error message carefully
3. Make sure you followed each step exactly
4. Try starting over from Step 3 (setting up Fuseki)

Remember: It's okay to make mistakes! Learning is part of the process. 🚀

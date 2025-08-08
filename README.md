<h1>GEP Mass-Create Automation Tool</h1>

<p>
  Automates creation of GEP supplier master workbooks from user templates plus reference (“master”) sheets.
  <br><strong>Impact:</strong> reduces ~16 hours of manual prep to ~5 minutes.
</p>

<p>
  <img alt="status" src="https://img.shields.io/badge/status-active-brightgreen">
  <img alt="python" src="https://img.shields.io/badge/python-3.9%2B-blue">
  <img alt="license" src="https://img.shields.io/badge/license-MIT-lightgrey">
</p>

<hr>

<h2 id="toc">Table of contents</h2>
<ul>
  <li><a href="#overview">Overview</a></li>
  <li><a href="#features">Features</a></li>
  <li><a href="#requirements">Requirements</a></li>
  <li><a href="#quick-start">Quick start</a></li>
  <li><a href="#configuration">Configuration</a></li>
  <li><a href="#usage-patterns">Usage patterns</a></li>
  <li><a href="#outputs">Outputs</a></li>
  <li><a href="#performance">Performance</a></li>
  <li><a href="#data-expectations">Data expectations</a></li>
  <li><a href="#troubleshooting">Troubleshooting</a></li>
  <li><a href="#roadmap">Roadmap</a></li>
  <li><a href="#contributing">Contributing</a></li>
  <li><a href="#license">License</a></li>
</ul>

<hr>

<h2 id="overview">Overview</h2>
<ul>
  <li>Ingests six user sheets: <code>Basic</code>, <code>Contact</code>, <code>TeamMember-GEP</code>, <code>Company</code>, <code>Purch</code>, <code>Bank</code>.</li>
  <li>Reads master/reference sheets (Org Entity, Category, Control Department, Region, Supplier Name Prefix, Classification, Trading Partner, Instruction Key, Sort Key, Planning Cash Management, Schema Group Vendor, Identification, Buyer User, Team Member, Recon Account, Payment Terms, Partner Bank Type, SwiftCode PE1/PE2, Country, Currency, State, Incoterms, WHT, statuses, Onboarding Type, Payment Method).</li>
  <li>Cleans, normalizes, maps, validates, and exports a GEP-ready master workbook plus batch XLSX files.</li>
</ul>

<h2 id="features">Features</h2>
<ul>
  <li>Deterministic cleaning: drop empties, cast to string, trim, standardized headers.</li>
  <li>Robust mapping: exact and substring VLOOKUP equivalents, case-insensitive.</li>
  <li>Field splitting: name/address into GEP field-length buckets.</li>
  <li>Cross-entity assembly: Org Entities, Payment Terms, Banking, Recon, Withholding Tax, Incoterms, LOB details.</li>
  <li>Validation reports for mismatches.</li>
</ul>

<h2 id="requirements">Requirements</h2>
<ul>
  <li>Python 3.9+</li>
  <li>Install from <code>requirements.txt</code> (pandas, numpy, openpyxl, xlsxwriter, tqdm, xlrd ≥ 2.0.1)</li>
</ul>

<h2 id="quick-start">Quick start</h2>

<pre><code class="language-bash"># 1) Clone &amp; install
git clone https://github.com/&lt;your-org&gt;/GEP-Mass-Create-Automation-Tool.git
cd GEP-Mass-Create-Automation-Tool
python -m venv .venv &amp;&amp; . .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 2) Point the script to your own paths
# edit INPUT_DIR, MASTER_DIR, OUTPUT_DIR to your local folders
vim src/mass_create_pipeline.py

# 3) Execute
python src/mass_create_pipeline.py
</code></pre>

<h2 id="configuration">Configuration</h2>

<table>
  <thead>
    <tr>
      <th>Option</th>
      <th>Location</th>
      <th>Default</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr><td><code>INPUT_DIR</code></td><td>top of <code>mass_create_pipeline.py</code></td><td><code>data/input</code></td><td>Folder containing user templates</td></tr>
    <tr><td><code>MASTER_DIR</code></td><td>same</td><td><code>data/master</code></td><td>Folder with reference sheets</td></tr>
    <tr><td><code>OUTPUT_DIR</code></td><td>same</td><td><code>data/output_sample</code></td><td>Where processed files are written</td></tr>
    <tr><td><code>BATCH_SIZES</code></td><td>same</td><td><code>[1, 10, 50, 100]</code></td><td>Subsequent chunks are 350 rows each</td></tr>
    <tr><td>Log level</td><td>env var <code>LOG_LEVEL</code></td><td><code>INFO</code></td><td>Set to <code>DEBUG</code> for verbose logging</td></tr>
  </tbody>
</table>

<h2 id="usage-patterns">Usage patterns</h2>

<table>
  <thead>
    <tr><th>Mode</th><th>Command / Action</th></tr>
  </thead>
  <tbody>
    <tr><td>One-off (CLI)</td><td><code>python src/mass_create_pipeline.py</code></td></tr>
    <tr><td>Jupyter analysis</td><td>Run <code>src/mass_create_pipeline.ipynb</code> cell-by-cell</td></tr>
    <tr><td>Scheduled (cron/Airflow)</td><td>Call the same script; exit code <code>0</code>=success, <code>1</code>=fail</td></tr>
  </tbody>
</table>

<h2 id="outputs">Outputs</h2>

<table>
  <thead>
    <tr><th>File / Folder</th><th>Purpose</th></tr>
  </thead>
  <tbody>
    <tr><td><code>Mass Create_&lt;timestamp&gt;.xlsx</code></td><td>Fully populated master workbook (GEP format)</td></tr>
    <tr><td><code>Batch_Output/</code></td><td>Segmented XLSX files for incremental upload</td></tr>
    <tr><td><code>*_mismatch.csv</code></td><td>Optional QA logs for rows that failed validation</td></tr>
  </tbody>
</table>

<h2 id="performance">Performance</h2>

<table>
  <thead>
    <tr><th>Dataset size</th><th>Runtime<sup>†</sup></th></tr>
  </thead>
  <tbody>
    <tr><td>1,000 vendors</td><td>~40 s</td></tr>
    <tr><td>10,000 vendors</td><td>~5 min</td></tr>
    <tr><td>20,000 vendors</td><td>~11 min</td></tr>
  </tbody>
</table>

<p><sup>†</sup> Intel i7-1165G7, 16 GB RAM. I/O (Excel read/write) dominates; CPU usage &lt; 30%.</p>

<h2 id="data-expectations">Data expectations</h2>

<details>
  <summary><strong>User input workbook</strong> (required columns per sheet)</summary>
  <ul>
    <li><strong>Basic</strong> — <code>VendorCode</code>, <code>Name</code>, <code>Prefix</code>, <code>LOB</code>, <code>Subcategory</code>, <code>ServiceLocation</code>, <code>Classification</code>, <code>ControlDepartment</code>, <code>Currency</code>, <code>Address1</code>, <code>Address2</code>, <code>City</code>, <code>State</code>, <code>CountryID</code>, <code>PostalCode</code>, <code>TradingPartner</code>, <code>InstructionKey</code>, <code>VatReg</code>/<code>NIK</code>/<code>NITKU</code>/<code>VatLama</code>, <code>Remark</code></li>
    <li><strong>Contact</strong> — <code>VendorCode</code>, <code>FirstName</code>, <code>LastName</code>, <code>emailAddress</code>, <code>contactStatus</code>, <code>designation</code>, <code>Telp1</code>, <code>Telp2</code>, <code>Fax</code>, <code>isPrimary</code></li>
    <li><strong>TeamMember-GEP</strong> — <code>VendorCode</code>, <code>emailAddress</code>, <code>roleName</code></li>
    <li><strong>Company</strong> — <code>VendorCode</code>, <code>CompanyCode</code>, <code>AccountGroup</code>, <code>ReconAccount</code>, <code>SortKey</code>, <code>CashMngmtGroup</code>, <code>PaymentMethod</code>, <code>PaymentTerms</code>, <code>WHTType</code>, <code>WHTCode</code></li>
    <li><strong>Purch</strong> — <code>VendorCode</code>, <code>PurchOrg</code>, <code>AccountGroup</code>, <code>PaymentTerms</code>, <code>Incoterms</code>, <code>Schma_Grp_Vendor</code>, <code>GRBased</code>, <code>SRVBased</code></li>
    <li><strong>Bank</strong> — <code>VendorCode</code>, <code>PaymentMethod</code>, <code>SwiftCode</code>, <code>PartnerBankType</code>, <code>CountryBank</code>, <code>BankKey</code>, <code>BankNumber</code>, <code>BankRef</code>, <code>IBAN</code>, <code>BeneficiaryName</code>, <code>BankName</code></li>
  </ul>
</details>

<p><em>Master/reference files</em> must include the sheets listed under <a href="#overview">Overview</a>.</p>

<h2 id="troubleshooting">Troubleshooting</h2>

<table>
  <thead>
    <tr><th>Symptom</th><th>Possible cause</th><th>Fix</th></tr>
  </thead>
  <tbody>
    <tr>
      <td><code>KeyError '&lt;column&gt;'</code></td>
      <td>Missing column in user template</td>
      <td>Ensure template includes all six required sheets and fields</td>
    </tr>
    <tr>
      <td>Mapping returns blank</td>
      <td>Value absent in reference sheet</td>
      <td>Add mapping to <code>/data/master/...</code> or update VLOOKUP list</td>
    </tr>
    <tr>
      <td>Excel file locked</td>
      <td>Output workbook open in Excel</td>
      <td>Close the file and rerun</td>
    </tr>
    <tr>
      <td><code>UnicodeEncodeError</code> on write</td>
      <td>Non-ASCII path characters (Windows)</td>
      <td>Move repo to a plain-ASCII path (e.g., <code>C:\Projects\gep\</code>)</td>
    </tr>
  </tbody>
</table>

<h2 id="roadmap">Roadmap</h2>
<ul>
  <li>Unit-test harness with <code>pytest</code> and sample fixtures</li>
  <li>Parquet/CSV intermediaries for faster reloads</li>
  <li>Docker image for zero-setup execution</li>
  <li>Optional Power BI / Tableau diagnostics on validation logs</li>
</ul>

<h2 id="contributing">Contributing</h2>
<ol>
  <li>Fork → create a feature branch → open a PR.</li>
  <li>Follow <code>black</code> and <code>isort</code> style guides.</li>
</ol>

<pre><code class="language-bash">pip install pre-commit
pre-commit install
</code></pre>

<p>Please include a short description and, when possible, at least one test under <code>/tests</code>.</p>

<h2 id="license">License</h2>
<p>MIT License. Built by the Finance Shared Service Division at Japfa to streamline supplier integration between GEP and SAP.</p>

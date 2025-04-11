# Flask Malware Scanner Demo

A web-based malware scanner demonstration application built with Python and Flask. This project showcases various malware detection techniques and provides a user interface for scanning files, quarantining threats, and managing quarantined items.

**Disclaimer:** This application is intended for educational purposes only and demonstrates basic malware detection concepts. It is **NOT** a production-ready, robust anti-malware solution and should **NOT** be used on critical systems or with real malware samples outside of controlled, isolated environments.

## Features

* **Web-Based UI:** User-friendly interface built with Flask and Bootstrap 5.
* **Multiple Detection Methods:**
    * **Known Hash Matching:** Detects files based on predefined MD5 and SHA-256 hashes listed in `config.ini`.
    * **YARA Rules:** Scans files against custom YARA rules defined in `rules.yar`. Includes EICAR test file detection by default.
    * **Suspicious Names:** Flags files with potentially risky extensions (e.g., `.exe`, `.vbs`) or common double extensions (e.g., `.pdf.exe`). Configurable via `config.ini`.
* **Archive Scanning:** Scans inside `.zip` archives for threats using the configured detection methods. Flags the entire archive if threats are found within.
* **Quarantine Management:**
    * Move detected threats (or archives containing threats) to a secure quarantine folder.
    * View currently quarantined items, including original path and detection details.
    * Permanently delete selected items from quarantine.
    * Restore selected items from quarantine back to their original location (with safety checks - prevents restoring archive contents and overwriting existing files).
* **Whitelisting:** Ignore known safe files based on their SHA256 hash or filename (configured in `config.ini`).
* **Configuration File:** Easily customize paths, detection rules (hashes, extensions), whitelists, and logging settings via `config.ini`.
* **Logging:** Records scan activity, detections, errors, and actions to a log file (`malware_detector.log` by default).

## Project Structure
malware_detector/
├── myenv/                  # Python virtual environment (if used)
├── quarantine/             # Default directory for quarantined files
│   └── .metadata.json      # Stores metadata about quarantined files
├── scan_target/            # Default directory to place files for scanning
├── templates/              # HTML templates for the web UI
│   ├── index.html
│   ├── results.html
│   ├── final.html
│   └── quarantine_view.html
├── app.py                  # Main Flask application file
├── config.ini              # Configuration file (!!! IMPORTANT TO EDIT !!!)
├── rules.yar               # YARA rules file
├── malware_detector.log    # Log file (created on run)
├── requirements.txt        # Python dependencies
└── README.md               # This file
## Prerequisites

* Python 3.x (Developed/Tested with Python 3.10+, adjust as needed)
* `pip` (Python package installer)
* Git (for cloning the repository)
* **For `yara-python`:** You might need `yara` itself and potentially C compiler tools installed on your system depending on your OS. Please refer to the [yara-python documentation](https://yara-python.readthedocs.io/en/stable/gettingstarted.html#installing-yara-python) for details if installation fails.

## Installation & Setup

1.  **Clone the Repository:**
    ```bash
    git clone <your-repo-url>
    cd malware_detector
    ```

2.  **Create and Activate Virtual Environment (Recommended):**
    * **macOS/Linux:**
        ```bash
        python3 -m venv myenv
        source myenv/bin/activate
        ```
    * **Windows:**
        ```bash
        python -m venv myenv
        myenv\Scripts\activate
        ```

3.  **Install Dependencies:**
    ```bash
    pip install -r requirements.txt
    ```
    *(You need to create `requirements.txt`. See below.)*

4.  **Create `requirements.txt`:**
    Create a file named `requirements.txt` in the `malware_detector` directory with the following content:
    ```txt
    Flask>=3.0.0
    yara-python>=4.5.0
    Werkzeug>=3.0.0  # Often comes with Flask
    Jinja2>=3.1.0    # Often comes with Flask
    click>=8.1.0     # Often comes with Flask
    itsdangerous>=2.2.0 # Often comes with Flask
    blinker>=1.8.0      # Often comes with Flask
    ```
    *(Adjust versions if necessary based on your installation)*

5.  **Configure `config.ini`:**
    * Open `config.ini`.
    * **Crucially:** Update `KnownMD5Hashes` and `KnownSHA256Hashes` with the actual hashes of files you want to detect specifically by hash.
    * Update `WhitelistedHashesSHA256` and `WhitelistedFileNames` if you have known safe files you want to ignore.
    * Verify `ScanDirectory`, `QuarantineDirectory`, `YaraRulesFile`, `LogFile`, and `QuarantineMetadataFile` paths are correct for your setup (defaults should work if directories are next to `app.py`).
    * Adjust `SuspiciousExtensions` or `DoubleExtensionTriggers` if needed.

6.  **Prepare Scan Directory:**
    * Ensure the `scan_target` directory exists.
    * Place files you want to scan inside `scan_target`. See "Testing" section below.

7.  **Prepare YARA Rules:**
    * Ensure `rules.yar` exists and contains valid YARA rules (the EICAR rule is included by default).

## Running the Application

1.  Make sure your virtual environment is activated.
2.  Navigate to the `malware_detector` directory in your terminal.
3.  Run the Flask development server:
    ```bash
    flask run
    ```
    *(If you encounter host binding issues on some systems, you might use `flask run --host=0.0.0.0` but be aware of security implications if your firewall is not configured.)*
4.  Open your web browser and go to `http://127.0.0.1:5000` (or the address provided by Flask).

## Usage

1.  **Scan:** Click the "Start Scan Now" button on the home page.
2.  **Results:** The application will scan the configured `scan_target` directory (including inside ZIP files) and display any detected threats on the results page.
3.  **Quarantine:** Check the boxes next to the detected items you want to quarantine and click "Quarantine Selected Files". Note that if threats are found inside a ZIP archive, the entire archive will be flagged and quarantined.
4.  **View Quarantine:** Click the "View Quarantine" link (usually in the header) to see all currently quarantined items.
5.  **Manage Quarantine:** On the Quarantine Management page, select items and:
    * Click "Restore Selected Files" to attempt moving them back to their original location (will fail safely for items originally in archives or if the destination already exists).
    * Click "Delete Selected Files" to permanently remove them from quarantine.

## Testing

To test the scanner's functionality:

1.  **EICAR:** Create a file named `eicar.com` in `scan_target` containing *only* the standard EICAR test string: `X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*`. This should be detected by the default YARA rule.
2.  **Hash Match:** Create files whose MD5 or SHA256 hash matches entries you added to `KnownMD5Hashes` or `KnownSHA256Hashes` in `config.ini`.
3.  **Suspicious Name:** Create empty files with names like `invoice.pdf.exe`, `script.vbs`, `installer.bat`.
4.  **ZIP Archive:** Create a `.zip` file containing some of the above test files and place the zip file in `scan_target`. The scanner should flag the zip file itself as "Contained Threats".
5.  **Whitelist:** Add the SHA256 hash or filename of a normally detected file (like `eicar.com`) to the whitelist sections in `config.ini`. Re-scan; it should now be ignored.
6.  **Benign Files:** Include normal `.txt`, `.jpg`, etc., files to ensure they are *not* detected (testing for false positives).

## <i class="bi bi-exclamation-triangle-fill text-danger"></i> Safety Warning

This is an **EDUCATIONAL DEMONSTRATION** and **NOT A REPLACEMENT FOR PROFESSIONAL ANTIVIRUS SOFTWARE**.

* **DO NOT** run this application on systems containing sensitive data without fully understanding the code and risks.
* **DO NOT** test this application with **REAL MALWARE** unless you are an expert working in a secure, isolated environment (like a virtual machine completely disconnected from all networks). Mishandling real malware can have severe consequences.
* The detection methods used are basic and can be easily bypassed by real-world threats.

## Future Improvements / Ideas

* Add more heuristic detection (String analysis, entropy).
* Implement a UI for managing configuration (`config.ini`).
* Add support for more archive types (RAR, 7z - may require external libraries).
* Implement scan progress indicators for the UI.
* Database storage for quarantine metadata instead of JSON.

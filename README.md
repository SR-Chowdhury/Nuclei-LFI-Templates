# Nuclei LFI Templates & Study Notes

A practical learning repository for understanding **Local File Inclusion (LFI)** and experimenting with custom [Nuclei](https://github.com/projectdiscovery/nuclei) templates in an authorised lab.

> **Safety:** Use these templates only on systems you own or have explicit permission to test. A large payload list can generate many HTTP requests.

## Contents

- [1. LFI fundamentals](#1-lfi-fundamentals)
- [2. How Nuclei works](#2-how-nuclei-works)
- [3. Repository structure](#3-repository-structure)
- [4. Setup on Kali Linux](#4-setup-on-kali-linux)
- [5. Run the templates](#5-run-the-templates)
- [6. Verify findings](#6-verify-findings)
- [7. Limitations](#7-limitations)
- [8. Prevention](#8-prevention)
- [9. Quick revision](#9-quick-revision)
- [References](#references)

## 1. LFI fundamentals

**Local File Inclusion (LFI)** is a web application vulnerability in which user-controlled input influences which local file an application reads or includes. If that input is handled insecurely, a user may access files that were not intended to be exposed.

**Path traversal** is a related technique that manipulates a file path with sequences such as `../` to move to a parent directory. It can lead to LFI or direct file disclosure, but the terms are not identical.

### Simple example

An application might accept a URL like:

```text
https://lab.example/page?file=about.html
```

The application uses the `file` parameter to choose a file. If it does not safely restrict that choice, the parameter might be manipulated to request an unintended file.

- **Linux example:** `/etc/passwd` contains account records; it does not contain users' passwords in plaintext.
- **Windows example:** `C:\Windows\win.ini` is a configuration file that may exist on Windows systems.

These are examples for authorised lab testing. A matching response string is evidence to investigate, not automatic proof of exploitability.

### LFI and related terms

| Term | Meaning |
|---|---|
| **LFI** | The application is tricked into reading or including a local server file. |
| **Path Traversal** | Input manipulates a path to access a location outside the intended directory. |
| **RFI (Remote File Inclusion)** | The application includes a remote resource as a file; it is distinct from LFI and depends on runtime/configuration. |
| **File Disclosure** | Sensitive file content is exposed, potentially due to LFI, path traversal, or misconfiguration. |

**Potential impact:** Depending on permissions and configuration, LFI may expose source code, configuration, credentials, or other sensitive data. **LFI does not automatically mean Remote Code Execution (RCE).** RCE is possible only under additional conditions.

## 2. How Nuclei works

Nuclei is a template-based vulnerability scanner. YAML templates define the requests to send and the response conditions (matchers) to evaluate.

1. Nuclei sends HTTP requests containing candidate file paths.
2. The server returns a response.
3. Matchers look for patterns associated with files such as Linux `/etc/passwd` or Windows `win.ini`.
4. A matching response is reported for further verification.

Nuclei automates checks; it cannot guarantee detection. Results depend on the target endpoint, parameter names, encodings, server behaviour, payloads, and matcher quality.

## 3. Repository structure

```text
Nuclei-LFI-Templates/
├── README.md
├── templates/
│   ├── LFI-template-1.yaml
│   └── LFI-template-2.yaml
└── payloads/
    └── lfi-payloads.txt
```

- **`LFI-template-1.yaml`** — hard-coded example requests and response matchers.
- **`LFI-template-2.yaml`** — a template intended to load candidate values from a payload file.
- **`lfi-payloads.txt`** — a large wordlist; using the entire list can take time and create substantial request volume.

## 4. Setup on Kali Linux

Check whether Nuclei is installed:

```bash
nuclei
```

Update the Nuclei community templates:

```bash
nuclei -update-templates
```

Clone this repository:

```bash
git clone https://github.com/SR-Chowdhury/Nuclei-LFI-Templates.git
cd Nuclei-LFI-Templates
```

Validate the custom templates before using them:

```bash
nuclei -validate -t templates/LFI-template-1.yaml
nuclei -validate -t templates/LFI-template-2.yaml
```

Validation checks template syntax and structure; it does **not** prove the detection logic is accurate.

## 5. Run the templates

Replace `https://YOUR-LAB-HOST` with a lab target you own or are authorised to assess.

### Template 1 — hard-coded examples

```bash
nuclei -u https://YOUR-LAB-HOST -t templates/LFI-template-1.yaml -rl 5
```

### Template 2 — file-based payloads

Run this command **from the repository root** so the relative payload path can resolve as intended:

```bash
nuclei -u https://YOUR-LAB-HOST -t templates/LFI-template-2.yaml -rl 5
```

**Important:** Template 2 uses `payloads/lfi-payloads.txt`. Confirm the payload file loads correctly in your installed Nuclei version and execution context. If Nuclei reports a payload-file error, inspect the error and validate the template rather than assuming the wordlist was used.

The examples assume specific paths and a `page` parameter. Real applications may use different endpoints or parameters, so adapt the template to an observed request in your authorised lab. Start with a small, reviewed payload list before using a large wordlist.

## 6. Verify findings

A match means the response satisfied a template matcher. It does **not**, by itself, prove exploitability or impact.

1. Inspect the exact request and response in your lab.
2. Check that the matched content came from the requested file, not from a cached page, example text, or unrelated application content.
3. Record the affected endpoint, parameter, evidence, and reproducible steps.
4. Confirm the finding manually before reporting it as a verified vulnerability.

No result does not prove the application is safe. The target may use different parameters, block traversal, return customised content, or contain issues not covered by the template.

## 7. Limitations

- **Hard-coded paths:** Template 1 assumes specific paths and parameters; adapt them for your lab.
- **Matcher false positives/negatives:** Generic signatures can match unrelated page content, while strict signatures can miss valid file output.
- **Severity:** File-read evidence alone does not always justify Critical severity; base severity on verified impact and context. These revised templates use `high` as a starting point, not a substitute for impact assessment.
- **Windows signatures:** Generic markers may appear in unrelated content.
- **Large payload list:** It can increase run time and request volume. Begin with a small list.
- **Payload-file behaviour:** Confirm that the referenced payload file is actually loaded.
- **No automatic RCE:** Reading a file does not by itself demonstrate code execution.

Treat these as learning templates, not production-validated detection rules. The payload list references an external collection; check its original licence and attribution terms before redistributing it.

## 8. Prevention

- Do not use raw user input directly as a filesystem path.
- Map user choices to a fixed allowlist of permitted file identifiers.
- Canonicalise and validate paths, then ensure the resolved path remains within the intended directory.
- Run the application with least-privilege filesystem permissions.
- Keep sensitive files outside publicly served directories.
- Log suspicious path requests and test file-handling code.

Input filtering alone is fragile. Prefer safe file-selection logic and strict allowlists.

## 9. Quick revision

**What is LFI?** A vulnerability where user-controlled input causes an application to read or include an unintended local server file.

**What does `../` mean?** It refers to the parent directory and is commonly used in path-traversal attempts.

**What is Nuclei?** A vulnerability scanner that sends requests defined in YAML templates and evaluates responses with matchers.

**What does a Nuclei match tell you?** The response met the template's detection condition. Manually verify it before confirming a finding.

**Does LFI always lead to RCE?** No. Impact depends on application behaviour, permissions, configuration, and other conditions.

**How do I validate a template?** Run `nuclei -validate -t path/to/template.yaml`. This checks validity, not detection accuracy.

## References

- [Nuclei documentation](https://docs.projectdiscovery.io/tools/nuclei/overview)
- [Nuclei template guide](https://docs.projectdiscovery.io/templates/introduction)
- [ProjectDiscovery generic Linux LFI template](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/vulnerabilities/generic/generic-linux-lfi.yaml)
- [ProjectDiscovery generic Windows LFI template](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/vulnerabilities/generic/generic-windows-lfi.yaml)
- [OWASP: Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

## Disclaimer

For education and authorised security testing only. Test systems you own or have explicit permission to assess, and follow the applicable laws, scope, and rules of engagement.

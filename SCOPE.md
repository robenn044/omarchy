Introduction
Omarchy looks forward to working with the security community to find vulnerabilities and keep our businesses and customers safe.
Program highlights
Gold Standard Safe Harbor
Adheres to Gold Standard Safe Harbor. 
AI Research Safe Harbor
Adheres to AI Research Safe Harbor. 
Platform Standards
Fully compliant with Platform Standards. 
Top Response Efficiency
This program's response efficiency is above 90%. 
1 day, 12 hours
Average time to first response
1 day, 18 hours
Average time to triage
N/A
Average time to bounty
1 day, 18 hours
Average time from submission to bounty
N/A
Average time to resolution
Rewards summary
Last updated on September 23, 2026. View changes 
Each severity lists the 90-day average bounty and the percentage of total resolved reports, if applicable.
—
$250
$750
$1,500
Our rewards are based on severity per CVSS (the Common Vulnerability Scoring Standard). Please note these are general guidelines, and reward decisions are up to the discretion of Omarchy.
Scope exclusions
Core Ineligible Findings are out of scope. 
Learn more 
Overview
Last updated on October 1, 2026. View changes 

Omarchy Security Vulnerability Disclosure Program
Thank you for your interest in helping secure the Omarchy project. We greatly appreciate the security research community's efforts to identify and responsibly disclose vulnerabilities.
Program Overview
This bug bounty program is designed to reward security researchers who responsibly discover and report vulnerabilities in our codebase.
Scope
This program covers vulnerabilities in the following repository:
In Scope: https://github.com/omacom/omarchy
Out of Scope
Upstream vulnerabilities: Any vulnerabilities originating from upstream dependencies are out of scope for monetary reward. We will work with researchers to ensure these are properly reported to upstream projects and will provide credit and acknowledgment. However, no bounty payment will be awarded for upstream issues.
Third-party services or websites
Documentation or example code
Social engineering or phishing attacks
Denial of service attacks
Brute force attacks
Third-party dependencies (unless the vulnerability is in how Omarchy specifically uses or integrates the dependency)
Severity Assessment: We use industry-standard CVSS scoring to determine severity levels. Factors considered include:
Real-world exploitability and attack complexity
Potential impact on users and the ecosystem
Whether the issue requires special configuration or user interaction
Availability of workarounds or mitigations
Disclosure Policy
Timeline: We ask that you allow us a reasonable time to patch vulnerabilities before public disclosure. A typical timeline is 90 days from confirmation.
Public Credit: We will publicly acknowledge researchers who report valid vulnerabilities on our security credits page (unless you prefer to remain anonymous).
Follow HackerOne's disclosure guidelines.
Submission Requirements
To be eligible for a bounty, your report must meet the following criteria:
Report Quality
Detailed reproduction steps: Provide clear, step-by-step instructions that allow us to reproduce the issue independently. Vague or incomplete reports will not be eligible for bounty consideration.
Proof-of-Concept (PoC): Include working proof-of-concept code or scripts that demonstrate the vulnerability. PoC artifacts should be provided in a clearly organized format.
Affected versions and components: Clearly specify which components and version(s) of Omarchy are affected.
Impact description: Explain the potential impact of the vulnerability (e.g., data exposure, code execution, authentication bypass).
Report Scope
One vulnerability per report (unless chaining multiple vulnerabilities is necessary to demonstrate impact)
Duplicate handling: When duplicates occur, only the first valid, fully reproducible report will be eligible for reward.
Meaningful findings: Reports that consist solely of automated scanner output, AI-generated findings without manual verification, or theoretical issues without working proof-of-concept will not be eligible for bounty.
Verify findings against default branch:** Findings should always be verified against the latest quattro branch, our default branch, before submitting a report. This helps avoid duplicate reports for vulnerabilities already fixed but not yet released.
What Does NOT Qualify
Vulnerabilities in dependencies without demonstrated exploitable impact in Omarchy's code
Issues that require the attacker to already have privileged access
Missing security headers or informational findings
Configuration recommendations without concrete security bypass
Issues in development-only code or tools
Reflected findings from automated tools without verification or working exploit
Program Rules
Detailed reports required: Please provide detailed reports with reproducible steps and PoC code. Reports that cannot be reproduced will not be eligible for reward.
Submit one vulnerability per report unless chaining is necessary to demonstrate impact
Duplicates: When duplicates occur, we award the first valid, reproducible report
No destructive testing: Do not damage, destroy, or disrupt any systems. Do not download or exfiltrate personal data.
Ask before testing unscoped areas: If you discover something that appears out of scope, ask the team before submitting
No social engineering: Social engineering, phishing, vishing, or smishing is prohibited
Good faith testing: Make good faith efforts to avoid privacy violations, data destruction, and service disruption
Respect account ownership: Only interact with accounts you own or with explicit permission
Responsible disclosure: Do not publicly disclose vulnerabilities without consent
No threats or abuse: Do not make threats against Omarchy team members or HackerOne staff
AI-generated reports: Do not submit AI-generated reports without first manually verifying the findings and confirming a working proof-of-concept
Response Timeline
We aim to meet the following Service Level Agreements:
Initial response: Within 5 business days of submission
Triage decision: Within 10 business days of submission
Bounty decision: Within 15 business days of triage
Patch availability: We will work to patch confirmed vulnerabilities in a timely manner; timeline varies based on complexity
Confidentiality & Coordination
By participating in this program, you agree to:
Keep all vulnerability details, proof-of-concept code, and communications confidential
Not disclose any information about the program or vulnerabilities to third parties without written consent from the Omarchy team
Maintain the confidentiality obligation even after the vulnerability is patched
Comply with all applicable laws and ethical standards
Vulnerability Examples & Focus Areas
We are particularly interested in:
We consider a bug a security vulnerability when it can be exploited to cross a meaningful security boundary: an untrusted or lower-privileged party gains access, permissions, or control they didn’t already have.
Code that could be more robust but does not cross a security boundary is an improvement rather than a security vulnerability. We may still merge a proposed fix and credit the reporter in our release notes.
Eligibility for our security credits page depends on whether a report identifies a confirmed security vulnerability, not on its severity.
How to Submit
Submit your vulnerability report directly through the HackerOne platform to this program. Include:
Title: A clear, concise title describing the vulnerability
Description: Detailed explanation of the issue
Reproduction Steps: Clear step-by-step instructions
Proof-of-Concept: Working code or detailed reproduction evidence
Impact: Explanation of the potential damage or exploitation
Affected Version(s): Specific version(s) impacted
Environment: Details of your testing environment
Attachments: PoC files, screenshots, or videos as applicable
Support & Questions
Questions about this program: Use the HackerOne platform to message the program team
Security contact: security@omarchy.org or the designated security contact in the repository
General support: See the Omarchy documentation
General bugs: For anything that isn’t a security vulnerability, please use the Omarchy issue tracker.
Acknowledgment & Credits
We will publicly acknowledge researchers who report valid vulnerabilities on our security credits page (unless you prefer to remain anonymous). Your contribution helps make Omarchy more secure for the entire community.
Thank you for helping keep Omarchy safe! We appreciate your responsible disclosure and collaboration in making our project more secure. Asset name
Type
Coverage
Max. severity
Bounty
Last update
Resolved Reports 
https://github.com/omacom/omarchy	Source code	
Critical
Eligible
Sep 24, 2026
0 (-)
